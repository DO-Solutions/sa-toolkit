# SA Toolkit

Shared knowledge base and command library for the DigitalOcean Solutions Architecture team.

Every SA clones this repo. Every engagement adds to it.

## Quick start

```bash
# 1. Clone
git clone git@github.com:digitalocean-sa/sa-toolkit.git ~/do/sa-toolkit

# 2. Set alias (add to ~/.zshrc or ~/.bashrc)
alias sa='cd ~/do/sa-toolkit && claude'

# 3. Enter workspace and verify
sa
# Type /qualify to test your first command
```

## What's in here

| Path | Purpose |
|------|---------|
| `CLAUDE.md` | SA context and instructions for Claude |
| `.claude/commands/` | Slash commands: /qualify, /profile, /cost, /poc-init, /capture |
| `skills/` | Reusable patterns: architectures, qualification, cost models, POC templates |
| `knowledge/` | Captured engagements, competitive intel, references |

## Slash commands

```
/qualify    Score technical fit and recommend product motion
/profile    Generate customer one-pager from raw notes
/cost       Build DO vs incumbent cost comparison
/poc-init   Scaffold POC repo with checklist and runbook
/capture    Store engagement results into knowledge base
```

## Contributing

Every SA is expected to contribute back after meaningful engagements.

```bash
# After a POC or significant discovery:
# 1. Run /capture in Claude to generate the structured entry
# 2. Review the generated files
# 3. Commit and push

git checkout -b capture/[customer-slug]-[date]
git add knowledge/ skills/
git commit -m "capture: [customer] — [one-line outcome]"
git push origin capture/[customer-slug]-[date]
# Open PR → SA lead reviews → merge
```

## Repo structure

```
sa-toolkit/
├── CLAUDE.md
├── README.md
├── .gitignore
├── .claude/
│   ├── settings.json
│   └── commands/
│       ├── qualify.md
│       ├── profile.md
│       ├── cost.md
│       ├── poc-init.md
│       └── capture.md
├── skills/
│   ├── architectures/
│   ├── qualification/
│   ├── cost-models/
│   └── poc-templates/
└── knowledge/
    ├── engagements/
    ├── competitive/
    └── references/
```

## Maintained by

DigitalOcean Solutions Architecture team. Questions → Slack `#sa-ai-toolkit`.
