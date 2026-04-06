# /engage — New Engagement Initialization

Initialize a new customer engagement: create the folder structure, populate engagement.yaml, and set up the git branch.

## Usage
```
/engage
```
Claude will ask for customer details and build everything from the answers.

## What Claude does

1. Prompts for engagement details (see inputs below)
2. Derives a `slug` from the customer name (lowercase, hyphens, no spaces)
3. Creates the folder structure under `projects/[slug]/`
4. Writes `engagement.yaml` with provided metadata
5. Creates `deliverables/` and `raw/` subdirectories
6. Creates and checks out a `customer/[slug]` git branch
7. Confirms what was created and suggests the next command

## Required inputs

Claude will ask for:
- **Customer name** — full name, e.g. "Acme Corp"
- **AE name** — who handed this off
- **One-line context** — what problem are they solving?
- **Stage** — qualifying / poc / technical-validation
- **Products in scope** (best guess, can update later)

## Output

```
projects/
└── acme-corp/
    ├── engagement.yaml
    ├── deliverables/        ← /commands write here
    └── raw/                 ← gitignored, for your working files
```

`engagement.yaml`:
```yaml
customer: Acme Corp
slug: acme-corp
sa: [from git config user.name]
stage: qualifying
fit_score: null
products: []
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
ae: [AE name]
notes: "[one-line context]"
```

Git branch created: `customer/acme-corp`

## Instructions for Claude

1. Start by asking for stage only, using a numbered selection list:

   ```
   New engagement — let's get the details.

   First, what stage are we at?

   1. Qualifying — early discovery, evaluating fit
   2. POC — building or running a proof of concept
   3. Technical Validation — validating a specific architecture or feature
   ```

2. Wait for the user to select. Once stage is confirmed, ask for the remaining four fields as a group:

   ```
   Got it — [Stage]. Now I need the specifics:

   1. Customer name — e.g. "Acme Corp"
   2. AE name — who handed this off?
   3. One-line context — what problem are they solving?
   4. Products in scope — best guess, e.g. DOKS, Managed Postgres, GPU Droplets

   Drop all four and I'll scaffold everything.
   ```

3. Once all four are provided, scaffold the engagement:
   - Derive the slug: lowercase, spaces → hyphens, strip special chars
   - Use `git config user.name` to populate the `sa` field
   - Use today's date for `created` and `last_updated`
   - Run these shell commands automatically:
     ```bash
     git checkout main
     git pull origin main
     mkdir -p projects/[slug]/deliverables
     mkdir -p projects/[slug]/raw
     # write engagement.yaml
     git checkout -b customer/[slug]
     git add projects/[slug]/engagement.yaml
     git commit -m "engage: [slug] — [one-line context]"
     git push -u origin customer/[slug]
     ```

4. Confirm with a summary:
   ```
   Engagement initialized: [Customer Name]
   Branch: customer/[slug]
   Folder: projects/[slug]/

   Next steps:
   → /profile   — generate customer one-pager
   → /qualify   — score technical fit
   ```

5. Do not run /profile or /qualify automatically — let the SA choose.
