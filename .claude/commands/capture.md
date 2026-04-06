# /capture — Engagement Results Capture

Close out an engagement: read all deliverables from the active project, extract what's reusable, write a structured entry to the knowledge base, and prepare the PR.

## Usage
```
/capture
```
Run after a POC, technical validation, or significant discovery. Claude reads the full project context and prompts for outcome.

## Active engagement detection

Claude will:
1. Check the current git branch for `customer/[slug]`
2. Load all files from `projects/[slug]/`:
   - `engagement.yaml`
   - `deliverables/customer-profile.md`
   - `deliverables/customer-qualification.md`
   - `deliverables/cost-comparison.md`
   - `deliverables/poc-plan.md`
3. Ask for outcome and key results
4. Generate the knowledge base entry
5. Optionally extract a reusable skill
6. Prepare the commit and PR instructions

## Claude asks for

- **Outcome**: closed-won / closed-lost / technical-validation / stalled / ongoing
- **Key result metric**: one number that captures the outcome (e.g., "900 Mbps throughput", "42% cost savings", "POC failed on latency")
- **What worked / what didn't**: honest debrief
- **Reusable pattern?**: anything worth extracting to `skills/`

## Output: knowledge base entry

Written to: `knowledge/engagements/YYYY-MM-DD-[slug].md`

```markdown
---
customer: [name or anonymized slug]
date: YYYY-MM-DD
sa: [name]
stage: [outcome]
products: []
outcome: [one sentence]
tags: []
---

# Engagement: [Customer] — [Use Case]

## Context
## What we did
## Results
## What worked
## What didn't work
## Reusable artifacts
## For the next SA who sees this customer type
```

## Output: skill extraction (if applicable)

Written to: `skills/[category]/[pattern-name].md`

Claude will ask: *"Is there a pattern here worth generalizing?"*
If yes, extract it and link it from the engagement entry.

## Git operations

After generating and writing all files, Claude runs the following automatically:

```bash
# Stage the capture output
git add knowledge/ skills/ projects/[slug]/engagement.yaml

# Commit
git commit -m "capture: [slug] — [one-line outcome]"

# Push the customer branch
git push origin customer/[slug]
```

After pushing, Claude outputs:
```
Captured and pushed: customer/[slug]

Next: open a PR from customer/[slug] → main
SA lead reviews and merges. Branch deleted after merge.

PR URL: https://github.com/DO-Solutions/sa-toolkit/compare/customer/[slug]
```

## Instructions for Claude
- Read ALL available deliverables before prompting — pre-fill everything you can
- Anonymize customer name if they haven't given reference permission — ask the SA
- Closed-lost captures are as valuable as wins — don't skip them
- Keep the "For the next SA" section tight: one paragraph, actionable advice only
- Cross-reference `knowledge/engagements/` for similar past engagements and note if this one confirms or contradicts them
- After writing all files, run the git commands above automatically — do not just print them
- If the push fails (e.g. auth error), print the commands for the SA to run manually
