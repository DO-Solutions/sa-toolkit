# /poc-init — POC Scaffold Generator

Scaffold a POC repository with a validation checklist, architecture diagram (text), and runbook.

## Usage
```
/poc-init
```
Provide: customer name, use case, primary products being tested, success criteria.

## Output

Claude will:
1. Generate a `README.md` for the POC repo
2. Create a `checklist.md` with pass/fail validation gates
3. Draft a `runbook.md` with setup steps
4. Suggest a directory structure

## POC README template

```markdown
# POC: [Use Case] — [Customer]

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

## Resources
- DO Console: https://cloud.digitalocean.com
- Runbook: [runbook.md](./runbook.md)
- Results: [results.md](./results.md)
```

## Checklist template

```markdown
# POC Validation Checklist: [Customer]

## Environment setup
- [ ] DO project created, team members invited
- [ ] VPC configured with appropriate CIDR
- [ ] SSH keys loaded
- [ ] Firewall rules: restrict to VPN/office IPs

## [Primary product] validation
- [ ] [test 1]
- [ ] [test 2]

## Performance
- [ ] Baseline established
- [ ] Load test run (target: X req/s)
- [ ] Latency p99 < X ms under load
- [ ] No errors under sustained load

## Failover (if HA required)
- [ ] Primary node killed, standby promoted in < 30s
- [ ] Application reconnected automatically
- [ ] No data loss confirmed

## Security
- [ ] No public exposure of internal services
- [ ] All credentials in environment variables or secrets manager
- [ ] Audit log reviewed

## Handoff
- [ ] Results captured in /knowledge/engagements/
- [ ] Cost comparison updated with actuals
- [ ] Champion briefed on findings
```

## Instructions for Claude
- Tailor checklist items to the specific products being tested
- Set realistic but challenging success criteria based on DO benchmarks
- Include a "day 0" runbook section covering account setup and access
- Flag any known limitations of DO products that might affect success criteria
