# Skill: Cloud Migration to DigitalOcean

Reusable patterns for migrating workloads from AWS, GCP, Azure, or on-prem to DigitalOcean.

## Migration types

| Type | Description | Complexity | When to use |
|------|-------------|------------|-------------|
| Lift-and-shift | Move VMs as-is to Droplets | Low | Fast migration, no refactoring budget |
| Re-platform | Swap managed services (RDS → Managed PG, EKS → DOKS) | Medium | Customer wants DO managed services |
| Re-architect | Redesign for DO primitives (App Platform, Spaces, etc.) | High | Greenfield mindset, long timeline |
| Hybrid cut-over | Run both clouds in parallel, migrate traffic gradually | Medium–High | Zero-downtime requirement |

---

## Phase template (use for any migration)

### Phase 0 — Discovery (1–2 weeks)
- Inventory: list all running services, databases, storage buckets, LBs, DNS records
- Dependencies: map internal service-to-service traffic
- Data volumes: GB of databases, object storage, block storage
- Traffic: peak requests/sec, egress GB/mo
- Compliance: any data residency or regulatory constraints?

### Phase 1 — Foundation (1–2 weeks)
- Provision DO project, VPC, firewall rules
- Set up DOKS or Droplet fleet (mirror sizing from source)
- Configure Spaces (if replacing S3/GCS)
- Set up Managed DB (if replacing RDS/CloudSQL)
- Validate network connectivity end-to-end

### Phase 2 — Data migration
- Databases: logical dump + restore, then replicate with logical replication until cut-over
- Object storage: `rclone` or `aws s3 sync` → Spaces
- Block storage: snapshot + transfer (or rsync for active volumes)
- Validate checksums after transfer

### Phase 3 — Application cut-over
- Deploy app stack to DO environment
- Run parallel (dual-stack) for N days minimum
- Switch DNS (TTL lowered to 60s beforehand)
- Monitor error rates for 24–48h
- Roll back plan: restore DNS TTL, flip back

### Phase 4 — Decommission
- Confirm all traffic on DO
- Scale down source cloud resources (don't delete yet — 30-day buffer)
- Transfer DNS ownership if applicable
- Document final architecture

---

## AWS → DO service mapping

| AWS Service | DigitalOcean Equivalent | Notes |
|------------|------------------------|-------|
| EC2 | Droplets | Match vCPU/RAM; DO pricing ~40–60% less |
| EKS | DOKS | DOKS is simpler; no node group complexity |
| RDS PostgreSQL | Managed PostgreSQL | Logical replication for live migration |
| RDS MySQL | Managed MySQL | mysqldump + replica lag approach |
| ElastiCache Redis | Managed Redis | DUMP/RESTORE or redis-cli --pipe |
| S3 | Spaces | S3-compatible API; rclone for bulk transfer |
| ELB/ALB | Load Balancers | DO LB supports HTTP/HTTPS/TCP/UDP |
| CloudFront | Spaces CDN | Limited to object storage CDN |
| Route 53 | Third-party DNS or DO DNS | DO DNS for simple use cases |
| VPC/Security Groups | VPC + Cloud Firewalls | DO firewalls are stateful |
| EFS | NFS on a Droplet | No managed NFS — self-managed only |
| Lambda | App Platform (background workers) | No FaaS native product |

---

## Data migration: Postgres live migration

```bash
# On source (AWS RDS)
pg_dump -h <rds-host> -U <user> -d <db> -F c -f dump.pgc

# Transfer
scp dump.pgc user@<do-droplet>:/tmp/

# On target (DO Managed PG)
pg_restore -h <do-pg-host> -U <user> -d <db> -F c /tmp/dump.pgc

# For near-zero downtime: use logical replication
# 1. Set wal_level = logical on source
# 2. CREATE PUBLICATION on source
# 3. CREATE SUBSCRIPTION on target
# 4. Wait for lag = 0, then cut over
```

---

## Object storage migration (S3 → Spaces)

```bash
# Install rclone
curl https://rclone.org/install.sh | bash

# Configure source (AWS) and target (DO Spaces)
rclone config  # add s3 remote and spaces remote

# Sync
rclone sync aws-remote:my-bucket spaces-remote:my-bucket \
  --progress \
  --transfers 32 \
  --checkers 16 \
  --s3-chunk-size 64M

# Verify count + size
rclone size aws-remote:my-bucket
rclone size spaces-remote:my-bucket
```

---

## DNS cut-over checklist

- [ ] Lower TTL to 60s at least 24h before cut-over
- [ ] Confirm new stack is production-ready and passing health checks
- [ ] Update A/CNAME records to DO IPs
- [ ] Monitor DNS propagation: `watch -n5 dig +short yourdomain.com`
- [ ] Watch error rate for 30 min post-cut-over
- [ ] Keep source stack running for 7 days minimum

---

## Common failure modes

| Failure | Root cause | Fix |
|---------|-----------|-----|
| Postgres restore fails | Version mismatch | Match major version; use pg_upgrade if needed |
| App can't reach Managed DB | VPC not configured | DB must be in same VPC, or trusted source IP added |
| Spaces 403 errors | ACL or CORS misconfigured | Check bucket policy; set CORS if browser uploads |
| DOKS pods can't pull images | Private registry auth | Create imagePullSecret with DO registry token |
| High latency post-migration | Wrong region | Ensure DB, DOKS, and LB are in same region |
| NLB health check failing | Source IP / DSR issue | Add local route: `ip route add to local <NLB_IP> dev eth0` |

---

## Cost model snapshot (AWS → DO, typical mid-size SaaS)

| Resource | AWS (on-demand) | DigitalOcean | Delta |
|----------|----------------|-------------|-------|
| 10x m5.large | ~$700/mo | ~$240/mo (10x s-2vcpu-4gb) | -66% |
| RDS PG db.m5.large Multi-AZ | ~$280/mo | ~$100/mo (Managed PG 4GB HA) | -64% |
| 5TB egress | ~$450/mo | ~$0 (included in bandwidth) | -100% |
| ALB | ~$25/mo | $12/mo | -52% |
| **Total** | **~$1,455/mo** | **~$352/mo** | **-76%** |

*Transfer costs are often the biggest surprise. Always calculate egress explicitly.*

---

## Related engagements

Add links here as captures come in:
- (none yet — first migration capture will populate this)
