# Skill: AWS Billing CSV → DO Cost Comparison

**Category:** Cost Models
**First used:** Arvore engagement (2026-04-06)
**Applies to:** Any prospect asking for an AWS vs DO cost comparison who can share their billing export

---

## When to use this

A customer shares their AWS billing CSV (from AWS Cost & Usage Report or the billing console export). You need to produce a service-level cost comparison without requiring them to self-report their infrastructure — which is always incomplete.

---

## Step 1: Get the right file

Ask for: **AWS Cost & Usage Report (CUR)** or the monthly billing CSV from the AWS Billing console.

- The CUR export is line-item granular (one row per usage period per resource)
- The billing console CSV is aggregated by invoice — easier to parse, slightly less detail
- Either works. If they have both, use CUR.

---

## Step 2: Parse the CSV

The file can be 500 KB – 10 MB. Use an agent or chunked reads.

**Key columns:**

| Column | Use |
|--------|-----|
| `RecordType` | Filter to `LinkedLineItem` only — avoids double-counting with `PayerLineItem` |
| `ProductName` | Group by this for service-level totals |
| `UsageType` | Contains instance type hints (e.g., `BoxUsage:m5.16xlarge`) |
| `ItemDescription` | Human-readable description of the charge |
| `TotalCost` | The billed amount — sum this per ProductName |
| `BillingPeriodStartDate` | Confirm the period you're analyzing |

**Aggregation query (conceptual):**
```
SELECT ProductName, SUM(TotalCost)
FROM billing_csv
WHERE RecordType = 'LinkedLineItem'
GROUP BY ProductName
ORDER BY SUM(TotalCost) DESC
```

**Instance type extraction:**
Look for `BoxUsage:<type>` patterns in `UsageType`, or `<type>` in `ItemDescription` for EC2.
For RDS: look for `InstanceUsage:db.<type>` in `UsageType`.

---

## Step 3: Map AWS services to DO equivalents

Use this mapping table as a starting point. Adjust based on actual workload.

| AWS Service | DO Equivalent | Fit | Pricing delta |
|-------------|---------------|-----|---------------|
| EC2 (general/small: t3, m5.2xl) | Droplets (General Purpose) | High | DO ~20–35% cheaper than RI |
| EC2 (memory: r5, x1) | Droplets (Memory-Optimized) | Medium | Similar price, max 32 vCPU on DO |
| EC2 (large: m5.16xl+) | Multiple Memory-Optimized Droplets | Medium | Requires horizontal re-arch |
| EC2 (ARM: c7g, m6g, t4g) | Droplets (AMD/Intel) | Medium | Re-benchmark required |
| RDS (PG/MySQL, Multi-AZ) | DO Managed PG/MySQL (standby) | High | DO ~50–60% cheaper |
| RDS (Aurora) | DO Managed PG | Medium | No Aurora serverless equiv |
| S3 | Spaces | High | $0.02/GB vs $0.023/GB; better egress pricing |
| CloudFront | Spaces CDN | High | $0.01/GB vs $0.085–0.12/GB egress |
| EKS | DOKS | High | $12/mo control plane vs $0.10/hr |
| ELB/ALB | DO Load Balancers | High | $12–20/mo vs AWS variable |
| ECR | DO Container Registry | High | $5–110/mo vs AWS per-GB |
| CloudWatch | DO Monitoring | High | Free on DO |
| VPC (NAT Gateway) | DO VPC | High | Free on DO — NAT GW charges eliminated |
| Route 53 | DO DNS | High | Free on DO |
| MWAA (Airflow) | Self-hosted on Droplet | Medium | $80–150/mo vs $500+/mo |
| Redshift | Self-hosted ClickHouse | Medium | $160–200/mo vs $400–600/mo |
| EMR (Spark) | Spark on DOKS | Low-Medium | Complex self-managed |
| Kinesis | DO Managed Kafka | Medium | $50+/mo; evaluate maturity |
| DynamoDB | DO Managed MongoDB / Redis | Medium | $15–50/mo |
| SQS | DO Managed Redis (queues) | Medium | $15–30/mo |
| GuardDuty | Cloud Firewalls + Datadog | Partial | Coverage gap |
| WAF | Cloudflare WAF | Partial | Not native to DO |
| AWS Config | None | None | Accept gap |
| Transfer Family | Self-hosted SFTP | Medium | $10–15/mo |
| Secrets Manager | DOKS Secrets / Vault | Medium | $0–15/mo |
| KMS | DO encryption at rest | Partial | $0 for at-rest; no HSM equiv |

---

## Step 4: Flags to raise automatically

Check for these in every bill:

- **NAT Gateway charges** (`NatGateway-Bytes` in UsageType) — always $0 on DO. Call this out explicitly.
- **ECR > $100/mo** — likely an image retention issue, not a cloud-pricing issue. Flag for cleanup regardless of migration.
- **CloudWatch > $200/mo** — Datadog or DO Monitoring replaces most of this.
- **ARM instances** (c7g/m6g/r6g/t4g in UsageType) — flag for re-benchmarking; DO has no Graviton equiv.
- **db.r5 / db.m5 Multi-AZ** — RDS is almost always overpriced vs DO Managed DB for these. Always lead with this in the pitch.
- **S3 > $1,000/mo** — check if it's storage or egress. If egress, DO Spaces is 9× cheaper. If storage alone, savings are modest.
- **Datadog via Marketplace** — stays regardless of cloud; don't promise savings on this line.

---

## Step 5: One-time migration costs to surface

Always include:

| Cost | Formula |
|------|---------|
| S3 data egress | `total_TB × 1024 × $0.09` |
| RI break-even | Needs RI inventory (`aws ec2 describe-reserved-instances`) |
| Engineering time | Estimate separately; not in this model |

---

## Output format

See `projects/arvore/deliverables/cost-comparison.md` for a complete example.

Standard table structure:
```
| Resource | AWS/mo | DO/mo | Notes |
```

Always include:
- Assumptions block at the top
- Total + annual rows
- "Services with no DO equivalent" section
- Caveats (especially egress cost and RI situation)

---

## Known pitfalls

1. **The comparison becomes negotiation ammo.** Customers who self-initiate a cost comparison are often already in a renewal conversation with AWS. Ask whether they're in a negotiation before handing over the deliverable — if yes, get discovery done first or you'll build AWS's bargaining chip for them.

2. **RI amortization masks true on-demand cost.** A $2,400 EC2 bill with m5.16xlarge instances means heavy RI coverage. Don't model DO at list price vs AWS at RI price — it looks worse than it is. Either get their RI inventory or model both at on-demand for a fair comparison.

3. **Brazil / São Paulo gap.** DO has no SP region. For Brazilian companies, confirm whether their workload is latency-sensitive for end users before investing in the comparison. If yes, it's likely a pass.

4. **ARM fleet.** If they run Graviton (c7g/m6g/r6g/t4g), their container images and compiled binaries may be ARM-only. DO is AMD/Intel. This can make the compute migration much harder than the pricing suggests.
