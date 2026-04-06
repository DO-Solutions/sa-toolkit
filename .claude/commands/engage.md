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

1. Ask for inputs one question at a time, in order. Wait for the answer before asking the next.
   - Start with: "What's the customer name?"
   - Then: "Who's the AE on this one?"
   - Then: "One-line context — what problem are they solving?"
   - Then: "What stage are we at? (qualifying / poc / technical-validation)"
   - Then: "Products in scope? Best guess is fine, we can update later."
2. Derive the slug: lowercase, spaces → hyphens, strip special chars.
3. Use `git config user.name` to populate the `sa` field.
4. Use today's date for `created` and `last_updated`.
5. Run these shell commands automatically:
   ```bash
   # Always start from a clean main
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
6. After creating, output a summary:
   ```
   Engagement initialized: [Customer Name]
   Branch: customer/[slug]
   Folder: projects/[slug]/

   Next steps:
   → /profile   — generate customer one-pager
   → /qualify   — score technical fit
   ```
7. Do not run /profile or /qualify automatically — let the SA choose.
