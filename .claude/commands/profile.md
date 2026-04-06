# /profile — Customer One-Pager Generator

Generate a structured customer profile from raw notes, call transcripts, Slack threads, or CRM data.

## Usage
```
/profile
```
Paste in raw notes after invoking. Claude will extract and structure the output.

## Output format

```
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
