# /capture — Engagement Results Capture

Store engagement outcomes into the knowledge base in a structured, searchable format.

## Usage
```
/capture
```
Run at the end of any meaningful engagement: discovery call, POC, technical validation, or competitive deal.

## What gets captured

Claude will prompt for any missing fields and generate:
1. An engagement entry in `knowledge/engagements/YYYY-MM-DD-[customer-slug].md`
2. Any reusable patterns extracted to `skills/`
3. A commit message suggestion

## Engagement entry template

```markdown
---
customer: [name or anonymized slug]
date: YYYY-MM-DD
sa: [your name]
stage: discovery | technical-validation | poc | closed-won | closed-lost
products: []
outcome: [one sentence]
tags: []
---

# Engagement: [Customer] — [Use Case]

## Context
[What problem were they solving? Where were they coming from?]

## What we did
[2–5 bullets: what the SA actually did]

## Results
[Quantified outcomes: latency, cost, uptime, migration time, etc.]

## What worked
[What patterns or approaches succeeded]

## What didn't work
[Honest callout of issues, limitations, gotchas]

## Reusable artifacts
- [link or filename of any skills/ entries created from this engagement]

## For the next SA who sees this customer type
[One paragraph of distilled advice]
```

## Instructions for Claude

1. Parse all context from the current conversation
2. Fill in every field you can from context — don't leave blanks you could reasonably infer
3. Prompt for: outcome (won/lost/ongoing), key result metric, any skills worth extracting
4. If a skill pattern is identified, create the skill file in `skills/` and link it from the engagement
5. After generating, output:
   ```
   Files to create:
   - knowledge/engagements/YYYY-MM-DD-[slug].md
   - skills/[category]/[pattern-name].md  (if applicable)

   Suggested commit:
   git add knowledge/ skills/
   git commit -m "capture: [customer slug] — [one-line outcome]"
   git push
   ```
6. Keep the engagement entry honest. Closed-lost captures are as valuable as wins.
