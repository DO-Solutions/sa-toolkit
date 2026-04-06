# /poc-init — POC Scaffold Generator

Scaffold a POC plan with validation checklist, architecture overview, and runbook — written into the active engagement's deliverables.

## Usage
```
/poc-init
```
Provide: use case, primary products being tested, success criteria. Claude pre-fills from qualification if available.

## Active engagement detection

Claude will:
1. Check the current git branch for `customer/[slug]`
2. If found, load `projects/[slug]/deliverables/customer-qualification.md` for context (products, risks, fit score)
3. Write output to `projects/[slug]/deliverables/poc-plan.md`
4. Update `engagement.yaml`: set `stage: poc`, `last_updated`

If not on a customer branch, Claude will ask which project this is for.

## Output: poc-plan.md

```markdown
# POC Plan: [Use Case] — [Customer]

**SA**: [name] | **Date**: YYYY-MM-DD | **Duration**: X weeks
**Products**: [list]

## Objective
[One sentence: what are we proving?]

## Success criteria
| Criteria | Target | Actual | Pass? |
|----------|--------|--------|-------|
| | | | ☐ |

## Architecture
[ASCII or Mermaid diagram]

## Timeline
| Week | Goal |
|------|------|
| 1 | Environment setup + baseline |
| 2 | [use case specific] |
| 3 | Load testing + results |
| 4 | Findings readout |

## Runbook
### Prerequisites
### Environment setup
### Validation steps
### Teardown

## Validation checklist
- [ ] Environment setup complete
- [ ] Baseline established
- [ ] Load test run
- [ ] Failover tested (if HA required)
- [ ] Security review done
- [ ] Results captured

## Resources
- DO Console: https://cloud.digitalocean.com
- Results: place outputs in projects/[slug]/raw/ during the POC
```

## Instructions for Claude
- Pull success criteria and product list from qualification deliverable if it exists
- Reference `skills/poc-templates/` and `skills/architectures/` for relevant patterns
- Set realistic but specific success criteria — vague criteria = unwinnable POC
- Include a "day 0" checklist covering DO account setup and access
- After writing the file, confirm: `Saved → projects/[slug]/deliverables/poc-plan.md`
- Remind SA: raw test output, configs, and scripts go in `projects/[slug]/raw/` (gitignored)
- Suggest running `/capture` after the POC completes
