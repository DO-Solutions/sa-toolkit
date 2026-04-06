# projects/

Active customer engagements. Each subfolder is one customer.

## Structure

```
projects/
└── [customer-slug]/
    ├── engagement.yaml      ← metadata: stage, score, SA, products (read by dashboard)
    ├── deliverables/        ← clean outputs from /commands — committed, shareable
    │   ├── customer-profile.md
    │   ├── customer-qualification.md
    │   ├── cost-comparison.md
    │   └── poc-plan.md
    └── raw/                 ← GITIGNORED — notes, logs, test configs, PII
```

## Rules

- `deliverables/` is committed. Keep it clean — no credentials, no raw PII.
- `raw/` is gitignored. Dump anything there freely.
- Each engagement lives on a `customer/[slug]` branch. Merge to `main` only after `/capture`.
- Run `/engage [customer name]` to initialize a new folder + branch automatically.

## engagement.yaml schema

```yaml
customer: Customer Name
slug: customer-slug
sa: SA Name
stage: qualifying        # qualifying | poc | technical-validation | closed-won | closed-lost
fit_score: null          # filled by /qualify
products: []             # filled by /qualify
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
ae: AE Name
notes: "One-line context"
```
