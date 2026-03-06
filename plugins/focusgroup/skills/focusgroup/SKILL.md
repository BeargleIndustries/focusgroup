---
name: focusgroup
description: AI-powered focus group reviews with deep protocols, advanced methodologies, quality control, and rich output formats
---

# Focus Group Skill

Multi-persona review system that dynamically generates diverse reviewer personalities to provide comprehensive, evidence-based feedback on any target. Supports deep review protocols, iterative methodologies, automated quality control, and rich output formats.

## Usage

```
/focusgroup https://example.com
/focusgroup ./src --deep
/focusgroup "a dating app for dogs" --delphi --full-report
/focusgroup https://mysite.com --panel startup --count 5
/focusgroup https://site.com --compare https://competitor1.com https://competitor2.com
/focusgroup https://mysite.com --walkthrough "sign up and make first purchase"
/focusgroup ./src --debate --strict --action-plan
/fg my-product --format json --html
```

## Arguments

| Argument | Description | Default |
|----------|-------------|---------|
| `<target>` | URL, file path, or idea description | (required) |
| `--count N` | Number of personas (3-15) | 10 |
| `--panel <type>` | Preset panel focus | (auto) |
| `--sequential` | Run reviews one at a time | parallel |
| `--parallel` | Run reviews in batches of 5 | (default) |
| `--format md\|json` | Output format | md |
| `--personas <file>` | Custom persona definitions file | (auto-generated) |
| `--out <dir>` | Output directory | .focusgroup/{timestamp} |
| `--deep` | Use full multi-phase review protocol per reviewer (8 phases for websites, 9 for codebases, 6 for ideas) | (standard) |
| `--shallow` | Skip deep site crawl, use homepage-only (NOT recommended for websites) | (deep) |
| `--delphi` | 3-round iterative Delphi method | (none) |
| `--debate` | Red Team vs Blue Team structured debate | (none) |
| `--walkthrough "goal"` | Cognitive walkthrough for a user goal | (none) |
| `--compare url1 [url2] [url3]` | Comparative review against competitors | (none) |
| `--strict` | Raise quality threshold to 75/100 | 60/100 |
| `--no-quality-check` | Skip automated quality validation | (enabled) |
| `--action-plan` | Generate prioritized action plan/backlog | (off) |
| `--backlog-format` | Action plan format: markdown\|csv\|github\|linear | markdown |
| `--swot` | Generate SWOT analysis matrix | (off) |
| `--risk-matrix` | Generate effort/impact matrix | (off) |
| `--html` | Generate self-contained HTML visual report | (off) |
| `--full-report` | Shorthand for --action-plan --swot --risk-matrix --html | (off) |

## Preset Panels

- **startup** - Product-market fit, growth, MVP, competitive analysis
- **enterprise** - Scalability, compliance, security, integration, vendor risk
- **accessibility** - WCAG, assistive tech, cognitive load, inclusive design
- **security** - Attack surface, auth, data protection, threat modeling
- **ux** - User journeys, visual design, interaction patterns, usability
- **performance** - Load times, bundle size, caching, scalability
- **content** - Messaging, tone, SEO, readability, audience targeting

## Input Types

- **Website**: URLs (http/https) — deep site crawl (all pages + linked properties) + browser tools + 8-phase protocol
- **Codebase**: File paths — Read, Grep, Glob, LSP, AST + optional deep 9-phase protocol
- **Product**: URL + local code — combines both approaches
- **Idea**: Plain text — web search + optional 6-phase research protocol

## Website Research (CRITICAL)

For website targets, the orchestrator MUST run a deep research phase BEFORE generating personas or launching reviewers. A homepage-only scrape produces wildly incomplete reviews.

### Research Phase (mandatory for all website targets)

1. **Launch the website-researcher agent** (`agents/probes/website-researcher.md`) to crawl the entire site:
   - Visit ALL internal pages (up to 25), not just the homepage
   - Follow outbound links to properties OWNED by the site owner (products, apps, research)
   - Read FULL text of all blog posts and articles
   - Run the technical JS probe
   - Output a comprehensive dossier to `.focusgroup/site-dossier.md`

2. **Pass the dossier to ALL downstream agents:**
   - The focus-director uses it to generate relevant personas
   - Each focus-reviewer receives it as their primary evidence source
   - The synthesis agent references it to verify reviewer claims

### Why This Matters

Without deep research:
- Reviewers critique absence of things that actually exist on subpages
- Products linked from the site are invisible to reviewers
- Published research/articles are missed entirely
- Reviews become shallow impressions of a homepage, not informed assessments

The dossier is the foundation of review quality. Never skip it.

## Methodology Modes

### Delphi (`--delphi`)
3-round iterative review: blind review → informed revision with anonymized group feedback → final positions. Produces an evolution trace showing how opinions shifted. Best for: building consensus on complex topics.

### Debate (`--debate`)
Red Team vs Blue Team with 3 rounds (Opening → Rebuttal → Closing) + Judge evaluation. Produces vulnerability report with defense ratings. Best for: adversarial quality testing, security reviews.

### Walkthrough (`--walkthrough "goal"`)
Each persona narrates their attempt to complete a specific user goal step by step, scoring friction per step. Produces a friction heatmap and drop-off funnel. Best for: UX evaluation, onboarding flows.

### Comparative (`--compare url1 url2`)
Each persona reviews all targets on identical criteria. Produces competitive matrix and gap analysis. Best for: competitive analysis, vendor selection.

Methodology flags are mutually exclusive — use only one per session.

## Quality Control

Reviews are automatically validated against quality standards:
- **Specificity check**: Catches vague phrases like "could be improved"
- **Evidence check**: Every claim must cite URLs, file paths, metrics, or quotes
- **Rating consistency**: Overall score must match individual area ratings
- **Completeness**: All template sections must meet minimum word counts
- **Hallucination guard**: Cross-references claims against probe data

Use `--strict` for higher quality threshold (75 vs 60). Use `--no-quality-check` to skip.

## Output Formats

### Action Plan (`--action-plan`)
Prioritized backlog with effort estimates (XS-XL), impact ratings, and measurable success criteria. Supports `--backlog-format markdown|csv|github|linear`.

### SWOT Matrix (`--swot`)
Strengths/Weaknesses/Opportunities/Threats analysis with cross-reference strategies.

### Risk/Impact Matrix (`--risk-matrix`)
2x2 effort-impact grid categorizing actions as Quick Wins, Strategic Bets, Fill-ins, or Avoid.

### HTML Report (`--html`)
Self-contained visual report with radar charts, rating bars, SWOT grid, and print-friendly styles.

### Full Report (`--full-report`)
Generates all four output types in one run.

## Output

All output goes to `.focusgroup/{timestamp}/`:
- `##-{slug}.md` — Individual persona reviews
- `synthesis.md` — Aggregate analysis with consensus, disagreements, ratings
- `meta.json` — Session metadata
- `probe-data.json` — Website diagnostic data (website/product targets)
- `action-plan.md` — Prioritized action plan (if requested)
- `swot.md` — SWOT analysis (if requested)
- `risk-impact-matrix.md` — Effort/impact matrix (if requested)
- `report.html` — Visual HTML report (if requested)
- Methodology-specific outputs (round directories, evolution traces, etc.)

## Advanced Examples

```
# Deep website review with full report
/focusgroup https://mysite.com --deep --full-report

# Delphi consensus with strict quality, 5 reviewers
/focusgroup ./src --delphi --strict --count 5

# Debate mode with action plan in GitHub issues format
/focusgroup https://app.com --debate --action-plan --backlog-format github

# Competitive analysis with HTML report
/focusgroup https://mysite.com --compare https://competitor1.com https://competitor2.com --html

# Walkthrough with custom personas
/focusgroup https://mysite.com --walkthrough "find and purchase a product" --personas examples/personas-accessibility.yaml
```

## Custom Personas

Create a YAML file with persona definitions:

```yaml
personas:
  - name: "Jane Smith"
    slug: "jane-smith"
    title: "Senior Product Designer, 12 years experience"
    archetype: "The Pragmatic Designer"
    expertise: ["UX design", "design systems", "user research"]
    perspective: "Form follows function — beauty without usability is decoration"
    review_priorities:
      - "Visual hierarchy and information architecture"
      - "Interaction patterns and affordances"
      - "Design system consistency"
    tone: "Direct but constructive, always suggests alternatives"
    rating_bias: "Scores high for clear user intent, low for style over substance"
```

See `examples/` for starter persona files: startup, security, investor, journalist, enterprise-buyer, accessibility.

## Magic Keywords

- "focusgroup", "focus group", "persona review", "multi-perspective", "panel review"
