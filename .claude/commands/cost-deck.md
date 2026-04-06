# /cost-deck — Customer Cost Comparison Spreadsheet

Generate a formatted .xlsx spreadsheet from the cost comparison deliverable.
This is the customer-facing version of `/cost` — same data, but in a spreadsheet
with formulas, charts, and formatting that works for presenting numbers to finance teams.

## Usage
```
/cost-deck
```

Requires: `deliverables/cost-comparison.md` must exist (run `/cost` first).

## What Claude does

1. **Detect active engagement** — read git branch, load `projects/[slug]/engagement.yaml`
2. **Read** `projects/[slug]/deliverables/cost-comparison.md`
3. **Parse** the cost comparison table — extract every line item with AWS cost, DO cost, and notes
4. **Generate** `projects/[slug]/deliverables/cost-comparison.xlsx` with:

### Sheet 1: Executive Summary
- Customer name, date, SA name (from engagement.yaml)
- Total AWS monthly / Total DO monthly / Monthly savings / Annual savings
- Savings percentage
- Top 3 biggest savings areas (by absolute $ difference)
- Top 3 risk areas (services with no DO equivalent or workaround needed)

### Sheet 2: Detailed Breakdown
Full line-item table with columns:
| A: Resource | B: AWS/mo ($) | C: DO/mo ($) | D: Savings ($) | E: Savings (%) | F: Notes |

- Column D = B - C (formula, not hardcoded)
- Column E = D / B (formula, with IFERROR for zero-division)
- Total row at bottom with SUM formulas
- Conditional formatting: green fill on savings > 50%, yellow on 20-50%, red on negative savings
- Freeze top row

### Sheet 3: Migration Risk
Services with no direct DO equivalent, pulled from the "No Direct DO Equivalent" section:
| A: AWS Service | B: Status | C: Recommendation | D: Current Cost | E: DO Alternative Cost |

### Formatting
- DigitalOcean blue (#0080FF) header rows, white text
- Dark navy (#031B4E) for sheet tabs
- Arial font throughout
- Currency format ($#,##0) for all cost cells
- Percentage format (0.0%) for savings columns
- Column widths auto-sized to content
- Print area set, landscape orientation for Sheet 2

## Output

```
projects/[slug]/deliverables/cost-comparison.xlsx
```

## Instructions for Claude

1. Read `deliverables/cost-comparison.md` — parse the markdown table into structured data
2. Use Python with openpyxl to generate the xlsx
3. Use Excel formulas (=SUM, =B2-C2, =IFERROR) — never hardcode calculated values
4. Apply formatting: DO blue headers, currency/percentage formats, conditional formatting
5. After generating, confirm: `Saved → deliverables/cost-comparison.xlsx`
6. Do NOT auto-commit — the SA will review and commit with other deliverables
