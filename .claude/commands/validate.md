# /validate — Internal Product Validation

Document an internal SA team validation — not tied to a specific customer.
Use this when you've tested a DO product capability, found a limitation, or run a benchmark
that would be useful in future customer conversations.

## Usage
```
/validate
```
No customer branch required. Can be run from `main` or a `validation/[topic]` branch.

## When to use vs /capture

- **`/capture`** — closing out a customer engagement
- **`/validate`** — documenting proactive internal testing (no customer attached)

## What Claude asks for

- **Product(s) tested**: what DO product(s) were exercised?
- **What question were you answering?**: one sentence — the hypothesis
- **What you did**: steps taken, infra provisioned, scripts run
- **Results**: quantified outcomes — numbers, pass/fail, limits found
- **Reproducible?**: is there a script or runbook that another SA could re-run?
- **Extract to skills/?**: is there a pattern worth adding to `skills/`?

## Output: validation entry

Written to: `knowledge/validations/YYYY-MM-DD-[product]-[topic].md`

```markdown
---
product: [DO product name(s)]
date: YYYY-MM-DD
sa: [name]
question: [one sentence — what were you testing?]
result: confirmed | refuted | partial | inconclusive
tags: []
---

# Validation: [Product] — [Topic]

## Hypothesis
[What were you trying to prove or disprove?]

## Setup
[Infrastructure provisioned, config used, scripts run]

## Results
[Quantified findings — numbers, limits, pass/fail table]

## Reproducibility
[Steps another SA would need to reproduce this — or link to script in skills/]

## Implications for customer conversations
[What does this mean when talking to customers? What can we now confidently claim?]

## Limitations / caveats
[What wasn't tested? What might change this result?]

## Reusable artifacts
[Link to any skills/ entries created from this validation]
```

## Git workflow

Validations don't need a customer branch. Two options:

**Option A — commit directly to main** (for quick, standalone validations):
```bash
git checkout main
git pull origin main
# run /validate, review output
git add knowledge/validations/ skills/
git commit -m "validate: [product] — [one-line finding]"
git push origin main
```

**Option B — validation branch** (for larger, multi-day validation work):
```bash
git checkout -b validation/[topic]
# run /validate, review output
git push origin validation/[topic]
# open PR → main
```

## Instructions for Claude

1. Ask all required inputs in one prompt — not one at a time
2. Pre-fill date and SA name from git config
3. After generating the file, confirm: `Saved → knowledge/validations/YYYY-MM-DD-[topic].md`
4. If a reusable pattern is found, create `skills/[category]/[pattern].md` and link it
5. Run git add + commit automatically:
   ```bash
   git add knowledge/validations/ skills/
   git commit -m "validate: [product] — [one-line finding]"
   git push
   ```
6. If on a feature branch, push that branch. If on main, push main.
