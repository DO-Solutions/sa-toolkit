# /cost — Cost Comparison Builder

Build a cost comparison between DigitalOcean and a customer's current (or proposed) infrastructure.

## Usage
```
/cost
```
Provide current workload spec: instance types, sizes, quantities, managed services, data transfer estimates.

## Active engagement detection

Claude will:
1. Check the current git branch for `customer/[slug]`
2. If found, load `projects/[slug]/engagement.yaml` and qualification deliverable for context
3. Write output to `projects/[slug]/deliverables/cost-comparison.md`
4. Update `engagement.yaml`: `last_updated`

If not on a customer branch, Claude will ask which project this is for.

## Required inputs

Claude will ask for any missing:
- Current provider (AWS / GCP / Azure / Hetzner / other)
- Instance types and quantities
- Managed databases (engine, size, HA?)
- Object storage (GB stored, transfer out/mo)
- Load balancers (count)
- Any other billable resources

## Output format

### Cost Comparison: [Customer] — [Provider] vs DigitalOcean

**Assumptions**: [list any estimates or gaps]

| Resource | [Provider] | DigitalOcean | Notes |
|----------|-----------|--------------|-------|
| Compute | $X/mo | $X/mo | |
| Managed DB | $X/mo | $X/mo | |
| Object storage | $X/mo | $X/mo | |
| Load balancer | $X/mo | $X/mo | |
| Data transfer | $X/mo | $X/mo | ← often the surprise |
| **Total** | **$X/mo** | **$X/mo** | |
| **Annual** | **$X** | **$X** | |
| **Savings** | | **$X (XX%)** | |

**Key callouts**:
- [e.g., "AWS data egress at $0.09/GB vs DO flat $0 included in bandwidth pool"]
- [e.g., "RDS Multi-AZ 2.3x more expensive than DO Managed PG with standby"]

**Caveats**:
- [any assumptions that could shift the numbers]

## Pricing references (as of 2025)
- Droplets: $6–$672/mo (1–64 vCPU)
- Managed PG: from $15/mo (1 vCPU/1GB) to $540+/mo (HA)
- Spaces: $5/mo for 250GB + $0.02/GB overage, $0.01/GB transfer
- Load Balancer: $12/mo
- Bandwidth: 1TB free per Droplet, $0.01/GB after

## Instructions for Claude
- Always show transfer costs explicitly — this is where DO wins most often
- If customer doesn't know their transfer volume, use $0.05/GB as a conservative estimate for AWS and flag it
- Round to nearest $5 for readability
- Note if DO doesn't have a direct equivalent (e.g., some specialized services)
- After writing the file, confirm: `Saved → projects/[slug]/deliverables/cost-comparison.md`
- Suggest running `/poc-init` as the next step if score warrants it
