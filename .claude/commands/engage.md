# /engage — New Engagement Initialization

Initialize a new customer engagement: create the folder structure, populate engagement.yaml, and set up the git branch.

## Usage
```
/engage
```

## Interaction format

Use a multi-step form to collect engagement details. Present one field at a time as a selection prompt. The user navigates through steps sequentially.

### Step 1: Stage
Present as a selection list:
- Qualifying — early discovery, evaluating fit
- POC — building or running a proof of concept
- Technical Validation — validating a specific architecture or feature

### Step 2: Customer name
Ask for free-text input:
- Example: "Acme Corp"

### Step 3: AE / TAM name
Ask for free-text input:
- Example: "Sarah Chen"

### Step 4: Context
Ask for free-text input:
- "What problem are they solving? (one line)"

### Step 5: Products in scope
Present as a multi-select list:
- DOKS (Managed Kubernetes)
- Managed Databases (Postgres / MySQL / Redis / Mongo / Kafka)
- Droplets (Compute)
- GPU Droplets (H100 / A100)
- App Platform
- Spaces (Object Storage)
- Load Balancers
- Other (type your own)

### Step 6: Confirm and create

Show a summary of all collected fields and ask for confirmation before scaffolding.

## What happens after confirmation

1. Derive slug from customer name: lowercase, spaces → hyphens, strip special chars
2. Get SA name from `git config user.name`
3. Use today's date for `created` and `last_updated`
4. Run these commands automatically:
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
5. Output:
   ```
   Engagement initialized: [Customer Name]
   Branch: customer/[slug]
   Folder: projects/[slug]/

   Next steps:
   → /profile   — generate customer one-pager
   → /qualify   — score technical fit
   ```
6. Do not run /profile or /qualify automatically — let the SA choose.

## engagement.yaml template

```yaml
customer: [Customer Name]
slug: [derived-slug]
sa: [from git config user.name]
stage: [selected stage]
fit_score: null
products: [selected products]
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
ae: [AE/TAM name]
notes: "[one-line context]"
```

## Output structure

```
projects/
└── [slug]/
    ├── engagement.yaml
    ├── deliverables/        ← /commands write here
    └── raw/                 ← gitignored, for your working files
```
