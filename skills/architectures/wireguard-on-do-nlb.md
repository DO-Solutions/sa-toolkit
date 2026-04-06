# Pattern: WireGuard UDP on DigitalOcean REGIONAL_NETWORK Load Balancer

**Source engagement:** Telnyx (2026-04-06)  
**Validated on:** DOKS + self-managed RKE2/Rancher  
**Products:** REGIONAL_NETWORK Load Balancer, Droplets / DOKS

---

## Use case

Customer wants to run WireGuard VPN pods on Kubernetes, with one or more pods fronted by a DigitalOcean Network Load Balancer. Common in:
- VPN-as-a-service products (one pod per end-customer)
- Telecom / networking companies using WireGuard for secure tunnels
- Any UDP-based service needing LB in front of K8s pods

---

## Architecture options

### Option A: Per-customer LB (1 LB per pod)

Each customer gets their own LB IP, their own pod, their own port.

```
Client A → LB_A (IP:51820) → wg-pod-a
Client B → LB_B (IP:51820) → wg-pod-b
```

- **Pros:** Clean network-level isolation, unique IP per customer
- **Cons:** $12/mo per customer, ~22s provisioning per new customer, LB account limits apply
- **When to use:** Customer hard requirement for unique IPs, or very small scale

### Option B: Shared gateway (1 LB, 1 pod, N peers)

All customers connect to the same pod and are differentiated by WireGuard public key.

```
Client A ─┐
Client B ─┤→ LB (IP:51820) → wg-gateway-pod (all peers configured)
Client C ─┘
```

- **Pros:** $12/mo flat regardless of customer count, simple ops
- **Cons:** Pod failure = all customers down; must add iptables for inter-peer isolation
- **When to use:** Cost-sensitive, no unique-IP requirement, manageable peer count

### Option C: Per-port per-pod + port 0 (recommended at scale)

Each customer gets their own pod and unique UDP port, all behind one LB. Port 0 (any-to-any UDP) forwards all ports without per-port rules.

```
Client A → LB (IP:32000) → wg-pod-0
Client B → LB (IP:32001) → wg-pod-1
Client C → LB (IP:32002) → wg-pod-2
(single LB rule: UDP:0 → UDP:0)
```

- **Pros:** $12/mo flat, cryptographic isolation per pod, unlimited customers per LB
- **Cons:** Pods see kube-proxy SNAT IP (not real client IP)
- **When to use:** Per-customer pod isolation needed, scale beyond a few hundred customers

---

## What doesn't work (and why)

**Single Service, multi-port, multi-pod:** Kubernetes Services cannot route different ports to different pods. All ports in a Service share the same endpoint list. A packet on port 32000 can land on any pod.

**Source-IP affinity with shared port:** WireGuard handshakes are encrypted for a specific server public key. If the LB routes the handshake to the wrong pod (wrong key), it's silently dropped. Source-IP affinity only pins after the first packet.

---

## Self-managed Kubernetes requirements (RKE2, Rancher, kubeadm, k3s)

These are NOT needed on DOKS — the Cloud Controller Manager handles them automatically.

### 1. NLB local route (critical — nothing works without this)

DO REGIONAL_NETWORK LBs use DSR. Incoming packets have the NLB IP as the destination. The kernel drops them unless there's a local route:

```bash
ip route add to local <NLB_IP> dev eth0
```

Persist via systemd:

```ini
[Unit]
Description=Add local route for NLB IP
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/sbin/ip route add to local <NLB_IP> dev eth0
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Reference: https://docs.digitalocean.com/products/networking/load-balancers/how-to/configure-droplets-for-nlb/

### 2. All nodes must be LB backends

Tag all cluster nodes (including control plane) as LB backends, or use pod affinity to restrict WireGuard pods to tagged worker nodes only.

### 3. Health check DaemonSet with hostNetwork

NodePort-based health checks don't work — response comes from the pod's node, not the node the NLB checked. Use a DaemonSet:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nlb-healthcheck
spec:
  selector:
    matchLabels:
      app: nlb-healthcheck
  template:
    metadata:
      labels:
        app: nlb-healthcheck
    spec:
      hostNetwork: true
      containers:
      - name: healthcheck
        image: python:3-alpine
        command: ["python3", "-m", "http.server", "30080"]
        ports:
        - containerPort: 30080
          hostPort: 30080
```

Configure the NLB to health check TCP 30080 on the backend nodes.

---

## WireGuard configuration

```
# Server
[Interface]
PrivateKey = <from K8s Secret>
ListenPort = <unique port>
MTU = 1360

# Client
[Interface]
PrivateKey = <client key>
Address = 10.13.N.2/32
MTU = 1360

[Peer]
PublicKey = <server public key>
Endpoint = <NLB_IP>:<port>
AllowedIPs = 10.13.N.0/24
PersistentKeepalive = 25
```

**MTU 1360:** Provides margin for WireGuard overhead + IP headers. Avoids fragmentation.  
**PersistentKeepalive 25:** Prevents the NLB from expiring the UDP session mapping (~60s idle timeout).

---

## Key persistence

Always store the server private key in a K8s Secret, not emptyDir. Pod restarts with emptyDir regenerate the key — all clients lose connectivity and must reconfigure.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: wg-server-key
type: Opaque
stringData:
  private.key: <base64-encoded-private-key>
```

---

## Operational notes

- **LB rule propagation:** ~19s for a new forwarding rule to take effect
- **Pod restart time:** ~5s for replacement pod to come up
- **Resource per WireGuard pod:** ~1m CPU, ~4 MiB RAM — not a bottleneck
- **Source IP:** Pods see kube-proxy SNAT IP. Use WireGuard public key for customer identification
- **NodePort range:** Default is 30000–32767. With ports starting at 32000, expand range if needed: `--service-node-port-range=20000-32767` on kube-apiserver

---

## Cost reference

| Model | LBs | Cost/mo |
|-------|-----|---------|
| Per-customer LB | N | $12 × N |
| Shared gateway or port 0 | 1 | $12 flat |
| Multi-region sharding | N | $12 × N |
