# /cost — Cost Comparison Builder

Build a cost comparison between DigitalOcean and a customer's current infrastructure by parsing their actual bill.

## Usage
```
/cost
```

## Input: Provider bill (PDF or CSV)

The SA should provide the customer's bill as a file in the project's `raw/` directory:

```
projects/[slug]/raw/bill.pdf
projects/[slug]/raw/bill.csv
projects/[slug]/raw/aws-bill-2026-02.csv
```

Any filename works. Claude will auto-detect the provider and format.

### Supported bill formats

| Provider | PDF | CSV | Notes |
|----------|-----|-----|-------|
| AWS | Yes | Yes (Cost Explorer export, billing CSV) | LinkedLineItem rows preferred |
| GCP | Yes | Yes (Billing export) | Look for SKU-level detail |
| Azure | Yes | Yes (Cost Management export) | Resource group level |
| Hetzner | Yes | — | Invoice PDF |
| Other | Yes | Yes | Claude will infer structure |

If no file is provided, Claude falls back to asking questions interactively.

## Active engagement detection

Claude will:
1. Check the current git branch for `customer/[slug]`
2. If found, load `projects/[slug]/engagement.yaml` and qualification deliverable for context
3. Look for bill files in `projects/[slug]/raw/` — read any PDF or CSV found
4. Write output to `projects/[slug]/deliverables/cost-comparison.md`
5. Update `engagement.yaml`: `last_updated`

If not on a customer branch, Claude will ask which project this is for.

## How Claude processes the bill

### Step 1: Parse the bill
- **CSV**: Read with standard CSV parsing. Identify columns for service name, usage type, cost, quantity. Group by service category.
- **PDF**: Extract text. Look for line items with service names and dollar amounts. Handle multi-page tables.

### Step 2: Categorize services
Map every line item into these standard categories:
- Compute (instances, VMs, containers)
- Managed Databases (RDS, Cloud SQL, etc.)
- Object Storage (S3, GCS, Blob)
- CDN / Content Delivery
- Load Balancers
- Kubernetes / Container Orchestration
- Networking (VPC, NAT, DNS, VPN)
- Monitoring / Logging
- Security (WAF, firewall, compliance)
- Data Transfer / Egress
- Message Queues / Streaming
- Caching
- Container Registry
- Support plan
- Third-party (Datadog, etc. — vendor-neutral, carry forward as-is)
- Other / Misc

### Step 3: Map to DigitalOcean equivalents
For each category, find the closest DO product and estimate the cost. Use the pricing references below.

### Step 4: Flag gaps
Services with no direct DO equivalent get marked with status:
- **Managed** — DO has a managed equivalent (e.g., Kinesis → Managed Kafka)
- **Self-host** — can run on Droplets but adds ops burden
- **Partial** — partner solution covers it (e.g., Cloudflare WAF)
- **No equiv** — accept the gap or keep on current provider

### Step 5: Instance mapping
When mapping compute instances, use this logic:
- Match by vCPU and RAM to closest Droplet size
- If instance has more vCPU/RAM than DO's largest in that tier, note it as a blocker
- ARM instances → map to AMD equivalent, flag for benchmarking
- GPU instances → map to DO GPU Droplets if available
- Spot/preemptible → no DO equivalent, map to regular Droplets and note the premium

## Output format

### Cost Comparison: [Customer] — [Provider] vs DigitalOcean

**Period analyzed:** [month/year from the bill]
**SA:** [from engagement.yaml]
**Date:** [today]

---

## Assumptions
[List every assumption: instance mapping estimates, single vs multi-AZ, vendor-neutral items carried forward, etc.]

---

## [Provider] Fleet Snapshot
[Summary of instance types, database instances, storage volumes found in the bill]

---

## Cost Comparison Table

| Resource | [Provider]/mo | DO/mo | Notes |
|----------|--------------|-------|-------|
| Compute | $X | $X | [instance mapping details] |
| Managed DB | $X | $X | |
| ... | | | |
| **Total** | **$X** | **$X** | |
| **Annual** | **$X** | **$X** | |
| **Annual savings** | | **$X (XX%)** | |

---

## Key Callouts
[Top 3-5 insights: biggest savings, surprise costs, egress traps]

---

## Services with No Direct DO Equivalent
| [Provider] Service | Status | Recommendation |
|--------------------|--------|----------------|

---

## Caveats
[Assumptions that could shift numbers: RI commitments, ARM fleet, region gaps, egress one-time cost]

---

## Next Steps
→ `/qualify` if not done yet
→ `/cost-deck` to generate the customer-facing xlsx
→ `/poc-init` to scope the POC

## Pricing references (as of 2025-2026)

### Compute (Droplets)
- Basic: $6–$96/mo (1–8 vCPU)
- General Purpose: $63–$1,008/mo (2–64 vCPU)
- CPU-Optimized: $42–$672/mo (2–32 vCPU)
- Memory-Optimized: $84–$1,344/mo (2–32 vCPU, up to 192 GB)
- GPU (H100): from $2.50/hr

### Managed Databases
- PostgreSQL/MySQL: from $15/mo (1 vCPU/1GB) to $1,680/mo (16 vCPU/64 GB, HA)
- MongoDB: from $15/mo
- Redis: from $15/mo
- Kafka: from $50/mo
- OpenSearch: from $15/mo

### Storage & CDN
- Spaces: $5/mo for 250 GB + $0.02/GB overage
- Spaces CDN: $0.01/GB transfer
- Block Storage: $0.10/GB/mo

### Networking
- VPC: free
- Load Balancer: $12/mo
- DNS: free
- Bandwidth: 1 TB free per Droplet, $0.01/GB after
- Reserved IP: free when attached

### Kubernetes (DOKS)
- Control plane: $12/mo per cluster
- Worker nodes: billed as Droplets

### Container Registry (DOCR)
- Starter: $0 (500 MB)
- Basic: $5/mo (5 GB)
- Professional: $20/mo (unlimited)

### Monitoring
- Built-in: free

## Instructions for Claude

1. First, scan `projects/[slug]/raw/` for any PDF or CSV files
2. If found: parse the bill automatically, don't ask item-by-item questions
3. If multiple files: ask which one is the bill (or use all if they're from the same period)
4. If no file found: tell the SA to drop their bill PDF/CSV in `raw/` and re-run, OR fall back to interactive questions
5. Always show transfer costs explicitly — this is where DO wins most often
6. If data is ambiguous, state the assumption rather than asking (SAs can correct in review)
7. Round to nearest $5 for readability
8. After writing: `Saved → projects/[slug]/deliverables/cost-comparison.md`
9. Suggest `/cost-deck` as the next step to generate the customer-facing xlsx
