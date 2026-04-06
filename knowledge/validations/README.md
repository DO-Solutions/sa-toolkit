# knowledge/validations/

Internal product validations — not tied to a specific customer engagement.

## What belongs here

Tests and validations run by the SA team to prove out DO product capabilities, find limitations,
or benchmark performance for use in future customer conversations. These are proactive, not reactive.

Examples:
- "How many WireGuard pods can one DO LB handle?"
- "What's DOKS upgrade latency at 50 nodes?"
- "Does Managed PG handle 10k concurrent connections?"
- "Spaces throughput under sustained load"

## How it's different from engagements

| | Customer engagement | Internal validation |
|--|--------------------|--------------------|
| Triggered by | AE/CSM request | SA team initiative |
| Customer attached | Yes | No |
| Branch model | `customer/[slug]` | `validation/[topic]` |
| Command to use | `/capture` | `/validate` |
| Stored in | `knowledge/engagements/` | `knowledge/validations/` |

## Naming convention

```
YYYY-MM-DD-[product]-[what-was-tested].md
```

Examples:
```
2026-04-06-wireguard-doks-nlb-at-scale.md
2026-03-15-managed-pg-connection-limits.md
2026-02-10-doks-upgrade-latency-50-nodes.md
```

## How validations feed the team

Validation results flow into two places:
1. `knowledge/validations/` — the raw findings, reproducible by any SA
2. `skills/` — extracted patterns, benchmarks, and gotchas referenced by /qualify and /poc-init

When Claude searches for past context during a customer engagement, it searches both
`knowledge/engagements/` and `knowledge/validations/` automatically.
