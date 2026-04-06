# /qualify — Technical Qualification Scoring

Score the technical fit for a DigitalOcean engagement and recommend a product motion.

## Usage
```
/qualify
```
Claude will prompt for required context if not already in the conversation.

## Required inputs
- Customer name / industry
- Current infrastructure (cloud provider, on-prem, or hybrid)
- Stated pain points or goals
- Scale indicators (users, requests/sec, data volume, team size)
- Budget signals (if any)
- Timeline / urgency

## Output format

### Technical Fit Score: XX/100

**Scoring breakdown**:
| Dimension | Score | Notes |
|-----------|-------|-------|
| Workload fit (DO sweet spot) | /25 | |
| Migration complexity | /20 | |
| Scale requirements | /20 | |
| Managed services match | /20 | |
| Competitive displacement opportunity | /15 | |

**Recommended motion**: [Land / Expand / Defend / Pass]

**Primary products**: [list top 2–3]

**Risks**: [bullet list]

**Recommended next step**: [one concrete action]

## Scoring guide

- **80–100**: Strong fit. Push to POC immediately.
- **60–79**: Moderate fit. Needs discovery to de-risk.
- **40–59**: Weak fit. Identify the blocker before investing.
- **<40**: Poor fit. Qualify out or pass to partner channel.

## Land vs Expand vs Defend vs Pass
- **Land**: New logo, clear workload fit, migration path exists
- **Expand**: Existing customer, new workload or product upsell
- **Defend**: Renewal at risk, competitive threat, need to demonstrate value
- **Pass**: Wrong scale, wrong workload, or better handled by AWS/GCP partner
