# SA Toolkit — Onboarding Guide

**Audience**: New DigitalOcean SA joining the AI-assisted operating model.
**Time to complete**: ~30 minutes.

---

## Prerequisites

Before starting, confirm you have:

| Requirement | Check |
|-------------|-------|
| Claude Code CLI installed | `claude --version` |
| Git access to `DO-Solutions/sa-toolkit` repo | `git ls-remote https://github.com/DO-Solutions/sa-toolkit.git` |
| DO org API key from SA lead | Check `#sa-ai-toolkit` Slack channel |
| Git configured with your identity | `git config --global user.name` |

**Install Claude Code** (if not already):
```bash
npm install -g @anthropic/claude-code
```

**Set org API key**:
```bash
export ANTHROPIC_API_KEY=<key-from-sa-lead>
# Add to ~/.zshrc or ~/.bashrc to persist
echo 'export ANTHROPIC_API_KEY=<key-from-sa-lead>' >> ~/.zshrc
```

**Configure git identity** (if not already):
```bash
git config --global user.name "Your Name"
git config --global user.email "you@digitalocean.com"
```

---

## Directory structure on your machine

The toolkit uses two separate root directories. Keep them separate.

```
~/do/
├── sa-toolkit/          ← shared repo (clone once, pull often)
│                           slash commands, skills, captured knowledge
│                           never has raw customer work in it
│
└── engagements/         ← your local customer work (never committed to shared repo)
    ├── acme-corp/
    ├── wireguard-test/
    └── [customer-slug]/
```

**`sa-toolkit/`** is the shared brain. Everyone clones it, everyone contributes back to it.

**`engagements/`** is your personal working directory. Scripts, configs, test output, raw notes. This is where you do the actual technical work. You pull from it when running `/capture`.

---

## Initial setup

### 1. Create your `do/` workspace

```bash
mkdir -p ~/do/engagements
```

### 2. Clone the toolkit

```bash
git clone https://github.com/DO-Solutions/sa-toolkit.git ~/do/sa-toolkit
```

### 3. Set the `sa` alias

Add to your `~/.zshrc` or `~/.bashrc`:
```bash
alias sa='cd ~/do/sa-toolkit && claude'
```

Reload:
```bash
source ~/.zshrc  # or source ~/.bashrc
```

### 4. Verify setup

```bash
sa
```

Claude starts in the `sa-toolkit` directory and loads `CLAUDE.md` automatically. Test a command:

```
/qualify
```

Claude should prompt you for customer context. If it does, you're set.

---

## How the toolkit works

```
[Slack / AE hands off]
        ↓
  Create engagement folder
  ~/do/engagements/[customer]/
        ↓
  Open sa-toolkit: sa
  Run /qualify → score fit
  Run /profile → customer one-pager
        ↓
  Do the technical work in the engagement folder
  (configs, scripts, test output)
        ↓
  Back in sa-toolkit:
  Run /capture → structured entry + skill extraction
        ↓
  git checkout -b capture/[customer]-[date]
  git commit + push + PR
        ↓
  Team merges → everyone has the knowledge
```

**You bring**: customer notes, call transcripts, test results, architecture requirements.
**Claude brings**: structure, scoring, cost math, and context from every prior engagement in `knowledge/`.

---

## End-to-end workflow: WireGuard validation example

A customer asks: *"Can we run WireGuard VPN on DigitalOcean to connect Droplets across regions?"*

You'll do ad-hoc testing, then capture it properly. Here's the full cycle.

---

### Step 1: Set up the engagement folder

```bash
mkdir -p ~/do/engagements/wireguard-test
cd ~/do/engagements/wireguard-test
# Start working: configs, scripts, notes go here
```

---

### Step 2: Qualify the opportunity

```bash
sa   # opens Claude in sa-toolkit
```

```
/qualify

Customer: Mid-size SaaS, 50 engineers
Current infra: AWS VPC + EC2, looking to cut costs
Problem: Need encrypted overlay network connecting Droplets across NYC and AMS regions
Scale: ~20 Droplets, <1 Gbps aggregate throughput
Budget: Flexible, moving to DO anyway
Timeline: 4 weeks to validate
```

**Expected output**: Fit score 0–100, recommended motion (Land / Expand / Defend / Pass), primary products, risks, next step.

---

### Step 3: Profile the customer

```
/profile

[paste Slack thread, call notes, or CRM snippet here]
```

Claude generates a structured one-pager. Review it, fill in any `TBD` fields, save it to your engagement folder.

---

### Step 4: Do the technical work

Work in your engagement folder, not the toolkit:

```bash
cd ~/do/engagements/wireguard-test
# provision Droplets, configure WireGuard, run iperf3, capture output
```

Use the toolkit for reference while you work:

```bash
sa
# In Claude:
# "Read skills/architectures/ and suggest a WireGuard hub-and-spoke setup for 2 regions"
```

As `skills/` fills up over time, Claude will draw on prior validated patterns automatically.

---

### Step 5: Build the cost comparison

```
/cost

Current: AWS
Resources: 20 EC2 t3.medium, VPN Gateway ($0.05/hr), 500GB/mo egress
Region: us-east-1
```

Claude outputs a side-by-side table. Review the assumptions, adjust if needed, share with customer.

---

### Step 6: Capture the engagement

Back in the toolkit, with your test results ready:

```bash
sa
```

```
/capture

Customer: [name or slug]
What we tested: WireGuard hub-and-spoke across NYC3 and AMS3
Results: [paste key metrics from your engagement folder]
Outcome: technical-validation passed
Reusable? Yes — WireGuard config pattern and benchmark numbers
```

Claude generates:
- `knowledge/engagements/YYYY-MM-DD-[customer].md` — structured engagement record
- `skills/architectures/[pattern].md` — if a reusable pattern was found (Claude will ask)

Review both files before committing.

---

### Step 7: Commit and push

```bash
cd ~/do/sa-toolkit
git checkout -b capture/wireguard-test-$(date +%Y-%m-%d)
git add knowledge/ skills/
git commit -m "capture: wireguard networking validation — [one-line outcome]"
git push origin capture/wireguard-test-$(date +%Y-%m-%d)
```

Open a PR → SA lead reviews → merges → the whole team has the pattern.

---

## Contributing back

Contributing is the expectation, not optional.

| When | What |
|------|------|
| After every POC or significant discovery | Run `/capture`, commit, PR |
| When you find a reusable pattern | Add to `skills/` |
| When a competitor comes up in a deal | Add to `knowledge/competitive/` |
| When pricing is wrong or stale | Update `skills/cost-models/` with new date |

**PR conventions**:
- Branch: `capture/[customer-slug]-[YYYY-MM-DD]` or `skill/[pattern-name]`
- Ask Claude to sanity-check the capture output before committing: *"Review this engagement entry for gaps or inaccuracies"*
- SA lead reviews PRs within 48h
- No perfect prose required — accurate and useful beats polished

**Anonymization**: Before committing, replace customer names with slugs if the customer hasn't given permission to reference them. `acme-corp` → `mid-market-saas-nyc` is fine.

---

## Keeping the toolkit current

```bash
# Pull latest before starting any engagement
cd ~/do/sa-toolkit
git pull origin main
```

Make this a habit. Other SAs are constantly adding patterns and engagement data.

---

## Troubleshooting

**Claude doesn't see my slash commands**
→ You're not in the right directory. Run `sa` (the alias), not just `claude`. Check: `ls ~/do/sa-toolkit/.claude/commands/`

**Claude isn't loading SA context**
→ `CLAUDE.md` must exist in the directory root. Check: `ls ~/do/sa-toolkit/CLAUDE.md`

**API key errors**
→ `echo $ANTHROPIC_API_KEY` — if empty, re-export and add to shell profile

**Git push rejected**
→ Make sure you're pushing a branch, not `main` directly. Use `capture/` or `skill/` branch prefix.

**Accidentally committed customer PII**
→ Tell the SA lead immediately. Do not try to fix it yourself with a force-push.

---

## Slack channels

| Channel | Purpose |
|---------|---------|
| `#sa-ai-toolkit` | Questions, issues, improvements to the toolkit |
| `#sa-engagements` | Share interesting findings from the field |
| `#sa-knowledge` | Flag PRs for review |
