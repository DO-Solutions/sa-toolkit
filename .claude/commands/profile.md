# /profile — Customer One-Pager Generator

Generate a structured customer profile from raw notes, call transcripts, Slack threads, or CRM data.

## Usage
```
/profile
```
Paste in raw notes after invoking. Claude will extract and structure the output.

## Active engagement detection

Claude will:
1. Check the current git branch for `customer/[slug]`
2. If found, load `projects/[slug]/engagement.yaml` for context
3. Write output to `projects/[slug]/deliverables/customer-profile.md`
4. Update `engagement.yaml` with any new info (stage, products, last_updated)

If not on a customer branch, Claude will ask which project this is for or suggest running `/engage` first.

## Output format

```markdown
# Customer Profile: [Name]
Last updated: YYYY-MM-DD | SA: [name]

## Summary
One paragraph: who they are, what they're building, why they're talking to DO.

## Company
| Field | Value |
|-------|-------|
| Industry | |
| Size | |
| Stage | Startup / Growth / Enterprise |
| Current cloud | |
| DO relationship | Prospect / Customer / Partner |

## Technical footprint
- **Primary workloads**:
- **Stack**:
- **Scale**:
- **Infra team size**:

## Pain points
1.
2.
3.

## DigitalOcean opportunity
- **Primary motion**: Land / Expand / Defend
- **Products**:
- **Deal size estimate**: $X/mo
- **Timeline**:

## Key contacts
| Name | Role | Relationship |
|------|------|-------------|
| | | Champion / Economic buyer / Technical eval |

## Open questions
-
-

## Next actions
- [ ]
- [ ]
```

## Instructions for Claude
- Extract all available info from the raw input
- Mark unknown fields as `TBD` — do not fabricate
- Flag contradictions or missing critical info at the bottom
- Keep the summary to 3 sentences max
- If scale data is missing, note it as a blocker for qualification
- After writing the file, confirm: `Saved → projects/[slug]/deliverables/customer-profile.md`
- Suggest running `/qualify` as the next step
