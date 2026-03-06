# focusgroup

A Claude Code plugin that runs AI-powered focus group reviews with deep review protocols, advanced methodologies, quality control, and rich output formats.

Point it at a website, codebase, product idea, or anything else — and get back detailed reviews from diverse perspectives, plus synthesis with confidence-scored findings, action plans, SWOT analysis, and more.

## What's New in v1.0

- **Deep Review Protocols** — 8-phase website exploration, 9-phase codebase analysis, 6-phase idea research with evidence-tier scoring
- **Methodology Modes** — Delphi (iterative consensus), Debate (red/blue team), Cognitive Walkthrough (friction mapping), Comparative (competitive analysis)
- **Quality Control** — Automated specificity checking, evidence validation, rating consistency, hallucination guard
- **Rich Outputs** — Action plans (markdown/CSV/GitHub/Linear), SWOT matrices, effort-impact matrices, self-contained HTML reports with SVG charts
- **Website Probe** — Single JavaScript diagnostic collecting meta, headings, forms, ARIA, performance, links, and security data
- **Confidence Scoring** — Every finding scored by agreement rate, evidence quality, and archetype diversity
- **Anti-Echo-Chamber** — Overlap detection, dissent spotlights, perspective gap analysis

## Install

```bash
claude plugin add BeargleIndustries/focusgroup
```

Or install locally:

```bash
git clone https://github.com/BeargleIndustries/focusgroup.git ~/.claude/plugins/local/focusgroup
```

## Quick Start

```
/focusgroup https://mysite.com
/focusgroup ./src
/focusgroup "a dating app for dogs"
```

## Usage

```
/focusgroup <target> [options]
```

### Core Options

| Option | Description | Default |
|--------|-------------|---------|
| `--count N` | Number of personas (3-15) | 10 |
| `--panel <type>` | Preset panel focus | auto |
| `--sequential` | Run reviews one at a time | parallel |
| `--format md\|json` | Output format | md |
| `--personas <file>` | Custom persona definitions (YAML) | auto-generated |
| `--out <dir>` | Output directory | `.focusgroup/{timestamp}` |
| `--deep` | Use full multi-phase review protocols | standard |

### Methodology Options

| Option | Description |
|--------|-------------|
| `--delphi` | 3-round iterative Delphi method with convergence tracking |
| `--debate` | Red Team vs Blue Team structured debate with Judge |
| `--walkthrough "goal"` | Cognitive walkthrough with friction scoring per step |
| `--compare url1 [url2] [url3]` | Comparative review against 1-3 competitors |

Methodology flags are mutually exclusive.

### Quality Options

| Option | Description | Default |
|--------|-------------|---------|
| `--strict` | Raise quality threshold to 75/100 | 60 |
| `--no-quality-check` | Skip automated quality validation | enabled |

### Output Options

| Option | Description |
|--------|-------------|
| `--action-plan` | Generate prioritized action plan/backlog |
| `--backlog-format <fmt>` | Format: `markdown\|csv\|github\|linear` |
| `--swot` | Generate SWOT analysis matrix |
| `--risk-matrix` | Generate effort/impact matrix |
| `--html` | Generate self-contained HTML visual report |
| `--full-report` | All four output types at once |

### Aliases

`/fg` and `/focus-group` also work.

## Target Types

The plugin auto-detects what you're reviewing:

- **Website** — URLs starting with `http://` or `https://`. Personas use browser tools to navigate, screenshot, and inspect the live site. With `--deep`, follows an 8-phase protocol covering discovery, navigation, interactive elements, performance, accessibility, SEO, mobile, and security.
- **Codebase** — Local file paths. Personas use file reading, search, LSP diagnostics, and AST pattern matching. With `--deep`, follows a 9-phase protocol covering structure, build health, static analysis, security, coverage, dead code, patterns, architecture, and CI/CD.
- **Product** — A URL paired with local code. Personas get both browser and code tools.
- **Idea** — Plain text descriptions. Personas use web search for market research. With `--deep`, follows a 6-phase protocol covering market sizing (TAM/SAM/SOM), competitor analysis, pain points, feasibility, business model, and regulatory landscape.

## Methodology Modes

### Delphi Method (`--delphi`)

Three rounds of iterative review:
1. **Round 1**: Blind review — each persona reviews independently
2. **Round 2**: Informed revision — each persona sees anonymized group summary, revises their positions
3. **Round 3**: Final positions — explicit "I maintain" or "I changed" statements

Produces an evolution trace showing how opinions converged or diverged across rounds. Best for building consensus on complex, ambiguous topics.

### Debate Mode (`--debate`)

Structured adversarial review:
1. Personas split into Red Team (critics) and Blue Team (defenders)
2. Three rounds: Opening Arguments → Rebuttals → Closing Arguments
3. An independent Judge scores each exchange on evidence quality, argument strength, and practical impact

Produces a vulnerability report rating each finding as Undefended, Weakly Defended, Strongly Defended, or Refuted. Best for security reviews and stress-testing assumptions.

### Cognitive Walkthrough (`--walkthrough "goal"`)

Task-completion narrative:
1. Goal decomposed into 5-15 discrete steps
2. Each persona narrates their attempt at every step: what they see, try, and experience
3. Friction scored 1-5 per step, with drop-off probability estimates

Produces a friction heatmap and drop-off funnel. Best for evaluating user flows, onboarding, and accessibility.

### Comparative Review (`--compare url1 url2`)

Multi-target analysis:
1. Each persona reviews ALL targets (primary + up to 3 competitors) on identical criteria
2. Side-by-side ratings per dimension

Produces a competitive positioning matrix and gap analysis. Best for competitive intelligence and vendor selection.

## Quality Control

Every review is automatically validated:

| Check | What It Does |
|-------|-------------|
| Specificity | Catches vague phrases ("could be improved", "seems good") |
| Evidence | Verifies every claim cites URLs, file paths, metrics, or quotes |
| Rating consistency | Ensures overall score matches individual area ratings |
| Completeness | Checks all sections are present with minimum word counts |
| Hallucination guard | Cross-references claims against probe data |

Reviews scoring below the threshold (60, or 75 with `--strict`) are automatically sent back for one revision with specific feedback.

## Output Formats

### Action Plan (`--action-plan`)
Converts all recommendations into a prioritized backlog with:
- Priority (P0-P3), effort (XS-XL), impact (High/Medium/Low)
- Measurable success criteria
- Source personas and confidence scores
- Formats: markdown (default), CSV, GitHub Issues, Linear

### SWOT Matrix (`--swot`)
Maps findings to Strengths/Weaknesses/Opportunities/Threats with cross-reference strategies (SO, WO, ST, WT).

### Effort-Impact Matrix (`--risk-matrix`)
2x2 grid categorizing actions as Quick Wins, Strategic Bets, Fill-ins, or Avoid with recommended execution order.

### HTML Report (`--html`)
Self-contained visual report with:
- Radar chart of scores across dimensions
- Persona rating bars with statistics
- SWOT and risk-impact grids
- Confidence-weighted findings
- Print-friendly styles

## Preset Panels

| Panel | Focus Areas |
|-------|-------------|
| `startup` | Product-market fit, growth, MVP scope, unit economics, competitive moat |
| `enterprise` | Scalability, compliance, security, integration, vendor risk, SLAs |
| `accessibility` | WCAG compliance, assistive tech, cognitive load, inclusive design |
| `security` | Attack surface, auth/authz, data protection, threat modeling |
| `ux` | User journeys, information architecture, visual design, interaction patterns |
| `performance` | Load times, bundle size, caching, database queries, scalability |
| `content` | Messaging, tone, SEO, readability, audience targeting |

## Custom Personas

Define your own review panel with a YAML file:

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

```
/focusgroup https://mysite.com --personas my-panel.yaml
```

Example persona files in `examples/`: startup, security, investor, journalist, enterprise-buyer, accessibility.

## Output Structure

```
.focusgroup/2026-03-05_14-30/
  01-sarah-chen.md              # Individual reviews
  02-marcus-rivera.md
  ...
  synthesis.md                  # Aggregate analysis
  meta.json                     # Session metadata
  probe-data.json               # Website probe results
  action-plan.md                # Prioritized backlog
  swot.md                       # SWOT analysis
  risk-impact-matrix.md         # Effort/impact matrix
  report.html                   # Visual HTML report
  report.json                   # (if --format json)
  round-1/                      # (Delphi methodology)
  round-2/
  round-3/
  delphi-evolution.md           # (Delphi methodology)
  red-team/                     # (Debate methodology)
  blue-team/
  judge-scores.md               # (Debate methodology)
  debate-report.md              # (Debate methodology)
  narratives/                   # (Walkthrough methodology)
  walkthrough-steps.md          # (Walkthrough methodology)
  friction-map.md               # (Walkthrough methodology)
  reviews/{persona}/{target}/   # (Comparative methodology)
  competitive-matrix.md         # (Comparative methodology)
  gap-analysis.md               # (Comparative methodology)
```

## How It Works

1. **Parse** — Extract target, flags, and options
2. **Detect** — Auto-classify target type (website, codebase, product, idea)
3. **Probe** — Run website diagnostic probe (website/product targets)
4. **Generate** — Opus-tier agent generates N diverse personas
5. **Review** — Each persona reviews using appropriate tools and protocols
6. **Validate** — Quality enforcer checks each review for specificity and evidence
7. **Synthesize** — Opus-tier agent produces aggregate analysis with confidence scoring
8. **Generate** — Output generators create action plans, SWOT, risk matrix, HTML report
9. **Report** — Summary displayed, full output saved to disk

## Advanced Examples

```bash
# Deep website review with full report output
/focusgroup https://stripe.com --deep --full-report

# Delphi consensus with strict quality, security panel
/focusgroup ./src --delphi --strict --panel security --count 5

# Red/blue team debate with action plan as GitHub issues
/focusgroup https://app.com --debate --action-plan --backlog-format github

# Competitive analysis with HTML report
/focusgroup https://mysite.com --compare https://comp1.com https://comp2.com --html

# UX walkthrough with accessibility personas
/focusgroup https://mysite.com --walkthrough "sign up and make first purchase" --personas examples/personas-accessibility.yaml

# Quick codebase review, no quality checks
/focusgroup ./src --count 3 --no-quality-check

# Idea validation with investor personas and SWOT
/focusgroup "AI-powered meal planning from fridge photos" --personas examples/personas-investor.yaml --swot --action-plan
```

## File Structure

```
focusgroup/
  .claude-plugin/plugin.json     # Plugin metadata
  commands/focusgroup.md          # Main orchestration command
  agents/
    focus-director.md             # Persona generation & synthesis (opus)
    focus-reviewer.md             # Individual reviews (sonnet)
    quality-enforcer.md           # Review quality validation (sonnet)
    probes/
      website-probe.md            # JavaScript diagnostic probe (haiku)
    protocols/
      website-review-protocol.md  # 8-phase website exploration
      codebase-review-protocol.md # 9-phase codebase analysis
      idea-research-protocol.md   # 6-phase idea research
      confidence-scoring.md       # Confidence scoring formula
    methodologies/
      delphi-protocol.md          # 3-round Delphi method
      debate-protocol.md          # Red/Blue team debate
      walkthrough-protocol.md     # Cognitive walkthrough
      comparative-protocol.md     # Multi-target comparison
    outputs/
      action-plan-generator.md    # Backlog generation (sonnet)
      swot-generator.md           # SWOT analysis (haiku)
      risk-impact-generator.md    # Effort/impact matrix (haiku)
      html-report-generator.md    # Visual HTML report (sonnet)
  skills/focusgroup/SKILL.md      # Skill metadata
  examples/
    personas-startup.yaml         # Startup-focused personas
    personas-security.yaml        # Security-focused personas
    personas-investor.yaml        # Investor personas
    personas-journalist.yaml      # Media/journalist personas
    personas-enterprise-buyer.yaml # Enterprise buyer personas
    personas-accessibility.yaml   # Accessibility advocate personas
  README.md
  CONTRIBUTING.md
  LICENSE
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding new protocols, methodologies, output formats, and persona presets.

## License

[MIT](LICENSE)
