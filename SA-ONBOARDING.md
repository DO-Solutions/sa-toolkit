# SA Toolkit — Onboarding Guide

**Audience**: New DigitalOcean SA joining the AI-assisted operating model.
**Time to complete**: ~30 minutes.

---

## Prerequisites

Before starting, confirm you have:

| Requirement | Check |
|-------------|-------|
| Claude Code CLI installed | `claude --version` |
| Git access to `digitalocean-sa/sa-toolkit` repo | `git ls-remote git@github.com:digitalocean-sa/sa-toolkit.git` |
| DO org API key from SA lead | Check `#sa-ai-toolkit` Slack channel |
| SSH key added to GitHub | `ssh -T git@github.com` |

**Install Claude Code** (if not already):
```bash
npm install -g @anthropic/claude-code
```

**Set org API key**:
```bash
export ANTHROPIC_API_KEY=<key-from-sa-lead>
# Add to ~/.zshrc or ~/.bashrc to persist
```

---

## Initial setup

### 1. Clone the repo

```bash
mkdir -p ~/do
git clone git@github.com:digitalocean-sa/sa-toolkit.git ~/do/sa-toolkit
```

### 2. Set the `sa` alias

Add to your `~/.zshrc` or `~/.bashrc`:
```bash
alias sa='cd ~/do/sa-toolkit && claude'
```

Reload:
```bash
source ~/.zshrc  # or source ~/.bashrc
```

### 3. Verify setup

```bash
sa
```

You should see Claude start in the `sa-toolkit` directory. It will load `CLAUDE.md` automatically.

Test a command:
```
/qualify
```

Claude should prompt you for customer context. If it does, you're set.

---

## How the toolkit works

When you type `sa`, you enter a Claude session that:
- Knows DigitalOcean products, positioning, and pricing
- Has 5 slash commands pre-loaded
- Can read and write to the `skills/` and `knowledge/` directories
- Maintains context across your whole engagement

**You bring**: customer notes, call transcripts, architecture requirements.
**Claude brings**: structure, scoring, cost math, and a memory of every prior engagement.

---

## First workflow: a real example

We'll walk through the WireGuard networking validation — a real engagement that lives in `knowledge/engagements/`.

### Scenario

A customer asks: *"Can we run WireGuard VPN on DigitalOcean to connect our Droplets across regions?"*

You've done some ad-hoc testing. Here's how to turn that into a proper engagement using the toolkit.

---

### Step 1: Qualify the opportunity

```
sa
```

In Claude:
```
/qualify

Customer: Mid-size SaaS, 50 engineers
Current infra: AWS VPC + EC2, looking to cut costs
Problem: Need encrypted overlay network connecting prod Droplets across NYC and AMS regions
Scale: ~20 Droplets, <1 Gbps aggregate throughput needed
Budget: Flexible, moving to DO anyway
Timeline: 4 weeks to validate
```

**Expected output**: Technical fit score, recommended motion (probably "Land" or "Technical Validation"), product list (Droplets, VPC, Cloud Firewall), risks.

---

### Step 2: Profile the customer

```
/profile

[paste call notes or Slack thread here]
```

Claude generates a structured one-pager with company info, technical footprint, open questions, and next actions.

Save the output:
```bash
# Claude will suggest writing it to:
knowledge/engagements/YYYY-MM-DD-[customer]-profile.md
```

---

### Step 3: Run the technical validation

For WireGuard specifically, the pattern is documented in:
```
skills/architectures/wireguard-overlay-network.md
```

Reference it in your session:
```
Read skills/architectures/wireguard-overlay-network.md and help me set up a WireGuard test
between two Droplets in NYC3 and AMS3. Generate a runbook and validation checklist.
```

Claude will generate a runbook using the established pattern. Run the test, collect results.

---

### Step 4: Build the cost comparison

```
/cost

Current: AWS
Resources: 20 EC2 t3.medium instances, VPN Gateway ($0.05/hr), 500GB/mo transfer out
Region: us-east-1
```

Claude outputs a table. Review it, adjust any assumptions, share with customer.

---

### Step 5: Capture the engagement

After the validation, run:
```
/capture
```

Claude will ask for:
- Outcome (pass/fail/ongoing)
- Key metric (throughput achieved, latency, cost delta)
- Any reusable patterns discovered

It generates:
- `knowledge/engagements/YYYY-MM-DD-[customer].md`
- Any new `skills/` entries if patterns were found
- A suggested commit message

**Commit and push**:
```bash
git checkout -b capture/wireguard-validation-2025-04-06
git add knowledge/ skills/
git commit -m "capture: wireguard networking validation — 900Mbps same-DC, pattern documented"
git push origin capture/wireguard-validation-2025-04-06
```

Open a PR → SA lead reviews → merge → everyone benefits.

---

## The WireGuard engagement as a worked example

The seed engagement for this toolkit is the WireGuard networking validation at:
```
knowledge/engagements/2025-04-06-wireguard-networking-validation.md
```

And the extracted reusable skill at:
```
skills/architectures/wireguard-overlay-network.md
```

Read both files to see the expected format and level of detail.

Key things to notice:
- The engagement entry answers "what did we find" — including what didn't work
- The skill answers "how to do this again" — with config, benchmarks, and a checklist
- They link to each other so future SAs can navigate from either direction

---

## Contributing back

Contributing is the expectation, not optional. Here's the contract:

| When | What |
|------|------|
| After every POC or significant discovery | Run `/capture`, commit, PR |
| When you find a reusable pattern | Add to `skills/` |
| When a competitor comes up | Add to `knowledge/competitive/` |
| When pricing is wrong or stale | Update `skills/cost-models/` with date |

**PR process**:
1. Branch: `capture/[customer-slug]-[YYYY-MM-DD]` or `skill/[pattern-name]`
2. Claude reviews your capture output before you commit — ask it to sanity check
3. SA lead reviews PRs, merges within 48h
4. No need for perfect prose — just accurate, useful content

---

## Troubleshooting

**Claude doesn't see my slash commands**
→ Make sure you're running `claude` from inside `~/do/sa-toolkit/`. The `.claude/commands/` directory must be present. Check: `ls .claude/commands/`

**Claude isn't loading SA context**
→ `CLAUDE.md` must exist in the directory root. Check: `cat CLAUDE.md | head -5`

**API key errors**
→ `echo $ANTHROPIC_API_KEY` — if empty, re-export and add to shell profile

**Committed something you shouldn't have**
→ Tell the SA lead immediately. Never commit customer names or PII without anonymizing first.

---

## Slack channels

| Channel | Purpose |
|---------|---------|
| `#sa-ai-toolkit` | Questions, issues, improvements |
| `#sa-engagements` | Share interesting findings |
| `#sa-knowledge` | Flag PRs for review |
