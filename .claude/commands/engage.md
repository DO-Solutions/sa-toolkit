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

1. Walk through each field one at a time as a wizard. Ask only one question per message. Wait for the answer before asking the next.

   **Step 1 — Stage** (numbered selection):
   ```
   New engagement — let's get the details.

   What stage are we at?

   1. Qualifying — early discovery, evaluating fit
   2. POC — building or running a proof of concept
   3. Technical Validation — validating a specific architecture or feature
   ```

   **Step 2 — Customer name** (after stage confirmed):
   ```
   Got it — [Stage].

   What's the customer name?
   ```

   **Step 3 — AE name:**
   ```
   Who's the AE on this one?
   ```

   **Step 4 — One-line context:**
   ```
   One-line context — what problem are they solving?
   ```

   **Step 5 — Products in scope** (numbered selection + free text):
   ```
   Products in scope? Select all that apply or type your own.

   1. DOKS (Managed Kubernetes)
   2. Managed Databases (Postgres / MySQL / Redis / Mongo)
   3. Droplets (Compute)
   4. GPU Droplets
   5. App Platform
   6. Spaces (Object Storage)
   7. Load Balancers
   8. Other — I'll type it
   ```

2. Once all five are collected, scaffold the engagement:
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
