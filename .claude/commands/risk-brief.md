# /risk-brief — Internal Qualification Risk Brief

Generate a formatted .docx internal brief from the qualification deliverable.
This is an internal-only document for SA leads and AEs — highlights risks, blockers,
and discovery gaps. Not customer-facing.

## Usage
```
/risk-brief
```

Requires: `deliverables/customer-qualification.md` must exist (run `/qualify` first).

## What Claude does

1. **Detect active engagement** — read git branch, load `projects/[slug]/engagement.yaml`
2. **Read** `projects/[slug]/deliverables/customer-qualification.md`
3. **Read** `projects/[slug]/deliverables/cost-comparison.md` (if exists, for financial context)
4. **Generate** `projects/[slug]/deliverables/qualification-brief.docx` with:

### Document Structure

#### Cover
- "Internal Qualification Brief" title
- Customer name, SA, date
- Fit score badge: GREEN (75+), YELLOW (50-74), RED (<50) with score prominently displayed
- "INTERNAL — DO NOT SHARE WITH CUSTOMER" watermark/notice

#### Section 1: Executive Summary (1 paragraph)
- One-liner: who is the customer, what do they want, what's the fit score
- Recommended motion (Land / Expand / Pass)
- Monthly savings opportunity (from cost comparison if available)

#### Section 2: Fit Score Breakdown
Table with 5 dimensions from qualification:
| Dimension | Score | Max | Assessment |

Color-coded: green cells for high scores, red for low

#### Section 3: Risk Matrix
The core of the document. Table format:
| Risk | Severity | Likelihood | Impact | Mitigation |

Severity color-coded: HIGH = red, MEDIUM = orange, LOW = green
Pull from the "Risks" section of the qualification

#### Section 4: What Fits / What Doesn't
Two side-by-side tables (or sequential):
- "Green light" — services that map cleanly to DO
- "Yellow/Red flags" — services with gaps or workarounds

#### Section 5: Discovery Questions
Numbered list of open questions from the qualification, with status:
- ❓ Open — not yet answered
- ✅ Answered — with the answer
- ⚠️ Blocking — can't proceed without this

#### Section 6: Recommended Next Steps
Numbered action items with owners (SA, AE, customer)

### Formatting
- DigitalOcean dark navy headers (#031B4E)
- Blue accent (#0080FF) for callouts
- Red (#E74C3C) for HIGH severity, orange (#E67E22) for MEDIUM, green (#27AE60) for LOW
- Arial font, US Letter, 1-inch margins
- Header: "DigitalOcean | Internal Qualification Brief"
- Footer: "CONFIDENTIAL — INTERNAL USE ONLY" + page number

## Output

```
projects/[slug]/deliverables/qualification-brief.docx
```

## Instructions for Claude

1. Read both deliverables (qualification required, cost comparison optional)
2. Extract structured data: risks, scores, fit/gap lists, discovery questions
3. Use docx-js (npm docx package) to generate the .docx
4. Apply DigitalOcean branding: navy headers, blue accents, color-coded risk cells
5. After generating, confirm: `Saved → deliverables/qualification-brief.docx`
6. Do NOT auto-commit — the SA will review and commit with other deliverables
