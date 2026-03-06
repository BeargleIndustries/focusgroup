---
description: Run an AI-powered focus group review with dynamically generated persona panels
argument-hint: <target> [--count N] [--panel TYPE] [--sequential|--parallel] [--format md|json] [--personas <file>] [--out <dir>] [--deep] [--delphi] [--debate] [--walkthrough "goal"] [--compare url1 url2] [--strict] [--no-quality-check] [--action-plan [--backlog-format FORMAT]] [--swot] [--risk-matrix] [--html] [--full-report]
aliases: [fg, focus-group]
---

# Focus Group Command

[FOCUS GROUP ACTIVATED - MULTI-PERSONA REVIEW MODE]

You are now running a Focus Group session. This spawns a dynamically-generated panel of reviewer personas to examine the user's target from multiple perspectives.

## User Input

{{ARGUMENTS}}

## Orchestration Protocol

Execute these phases in order. Announce progress at each phase boundary.

---

### Phase 1: Parse Input

Extract from `{{ARGUMENTS}}`:

- **target**: Everything before flags — a URL, file path, or quoted text
- **count**: `--count N` (default: 10, valid: 3-15, clamp if outside)
- **panel**: `--panel <type>` (default: auto-detect)
- **mode**: `--sequential` or `--parallel` (default: parallel)
- **format**: `--format md|json` (default: md)
- **personas_file**: `--personas <path>` (default: none)
- **out_dir**: `--out <dir>` (default: `.focusgroup/{YYYY-MM-DD_HH-MM}`)
- **deep**: `--deep` (default: false — enables full protocol phases)
- **methodology**: exactly one of `--delphi`, `--debate`, `--walkthrough "goal"`, `--compare url1 [url2] [url3]` (default: none)
- **walkthrough_goal**: quoted text after `--walkthrough`
- **compare_targets**: 1-3 non-flag args after `--compare` (warn and truncate if >3)
- **strict**: `--strict` (default: false — raises quality threshold to 75)
- **no_quality_check**: `--no-quality-check` (default: false)
- **action_plan**: `--action-plan` or `--backlog` (default: false)
- **backlog_format**: `--backlog-format markdown|csv|github|linear` (default: markdown)
- **swot**: `--swot` (default: false)
- **risk_matrix**: `--risk-matrix` (default: false)
- **html_report**: `--html` (default: false)
- **full_report**: `--full-report` (expands to action_plan + swot + risk_matrix + html_report)

Methodology flags are mutually exclusive. If >1 set: `[FOCUS GROUP] Error: Only one methodology flag allowed. Choose one of: --delphi, --debate, --walkthrough, --compare`

---

### Phase 2: Detect Input Type

| Check | Input Type |
|-------|------------|
| Starts with `http://` or `https://` | `website` |
| Path exists on filesystem (`test -e`) | `codebase` |
| Contains both a URL and a local path | `product` |
| None of the above | `idea` |

Announce: `[FOCUS GROUP] Target: {target} | Type: {type} | Panel: {panel} | Count: {count} | Mode: {mode}`

---

### Phase 2.5: Protocol & Probe Resolution

Select protocol paths based on input type:

| Input Type | Protocol Path(s) |
|------------|-----------------|
| website | `agents/protocols/website-review-protocol.md` |
| codebase | `agents/protocols/codebase-review-protocol.md` |
| idea | `agents/protocols/idea-research-protocol.md` |
| product | Both website and codebase protocols |

**For website/product targets:** Run the website probe ONCE before reviews begin:

```
Agent(subagent_type="focusgroup:website-probe", model="haiku", prompt="
TARGET: {target}
Write probe results to: {out_dir}/probe-data.json
")
```

Read `{out_dir}/probe-data.json` — this data will be passed to each reviewer.

Select methodology protocol path if a methodology flag is set:

| Flag | Methodology Path |
|------|-----------------|
| `--delphi` | `agents/methodologies/delphi-protocol.md` |
| `--debate` | `agents/methodologies/debate-protocol.md` |
| `--walkthrough` | `agents/methodologies/walkthrough-protocol.md` |
| `--compare` | `agents/methodologies/comparative-protocol.md` |

Announce: `[FOCUS GROUP] Protocol: {protocol_path} | Methodology: {methodology or 'standard'} | Deep: {yes/no}`

---

### Phase 3: Setup Output Directory

```bash
mkdir -p {out_dir}
```

Announce: `[FOCUS GROUP] Output: {out_dir}/`

---

### Phase 4: Generate Personas

**If `--personas <file>` was provided:** Read the YAML file and use those persona definitions. Skip generation.

**Otherwise:** Spawn ONE opus agent to analyze the target and generate personas:

```
Agent(subagent_type="focusgroup:focus-director", model="opus", prompt="
[PERSONA_GENERATION]
You are a Focus Group Director assembling a review panel.
TARGET: {target} | INPUT TYPE: {input_type} | PANEL: {panel or 'auto'} | COUNT: {count}
{If panel specified: Focus on {panel} concerns with some diversity.}
{If auto: Generate diverse panel covering all important dimensions for this target.}
Analyze the target thoroughly first (fetch pages, scan code, research domain).
Then generate exactly {count} personas using [PERSONA:NN]...[/PERSONA] format.
")
```

Parse persona definitions. If fewer than requested, note the gap but continue.
Announce: `[FOCUS GROUP] Generated {N} personas for review panel`

---

### Phase 5: Execute Reviews

**If a methodology flag is set:** Read the methodology protocol at {methodology_path}. Follow its instructions round by round. The command is the sole agent spawner — execute each round's reviewer spawn instructions directly.

Key rules: Spawn agents as `focusgroup:focus-reviewer` model="sonnet". Between rounds, aggregate per protocol. Write inter-round outputs to protocol-specified paths. Pass protocol_path for deep reviews. After all rounds, proceed to Phase 5.5.

**If NO methodology flag:** Run standard reviews as follows.

Build tool-routing instructions based on input type:

**For `website`:**
Tools: `mcp__claude-in-chrome__navigate`, `mcp__claude-in-chrome__read_page`, `mcp__claude-in-chrome__get_page_text`, `mcp__claude-in-chrome__javascript_tool`, `WebFetch`, `WebSearch`. USE THESE TOOLS — do not fabricate observations.

**For `codebase`:**
Tools: `Read`, `Grep`, `Glob`, `Bash`, `mcp__plugin_oh-my-claudecode_omc-tools__lsp_diagnostics`, `mcp__plugin_oh-my-claudecode_omc-tools__lsp_diagnostics_directory`, `mcp__plugin_oh-my-claudecode_omc-tools__ast_grep_search`. USE THESE TOOLS — do not fabricate observations.

**For `product`:** Include BOTH website and codebase tool instructions.

**For `idea`:**
Tools: `WebSearch`, `WebFetch`. USE THESE TOOLS to ground your review in real data.

**Parallel mode (default):** Split personas into batches of 5. For each batch, spawn all simultaneously with `run_in_background: true`.

For each persona:
```
Agent(subagent_type="focusgroup:focus-reviewer", model="sonnet", run_in_background=true, prompt="
[FOCUS_REVIEW:{NN}]
YOU ARE: {persona.name} | {persona.title} | {persona.archetype}
Expertise: {persona.expertise} | Perspective: {persona.perspective}
Tone: {persona.tone} | Rating Bias: {persona.rating_bias}
TARGET: {target} | INPUT TYPE: {input_type}

{tool_routing_instructions}

{If deep or methodology is set:}
PROTOCOL LOADING (MANDATORY):
Read the protocol at {protocol_path} before starting. Follow every phase in order.
{If product: Also read {second_protocol_path}.}

{If website/product and probe data exists:}
PROBE DATA: {paste probe-data.json content}
Use this as evidence. Do not re-run the probe.

REVIEW INSTRUCTIONS:
- Stay in character as {persona.name} throughout
- Use tools to thoroughly examine the target
- Focus on 3 priority areas: {persona.review_priorities[0-2]}
- Provide SPECIFIC EVIDENCE for every claim (URLs, file paths, line numbers, quotes)
- Rate each priority 1-5 and overall 1-10

Write review to: {out_dir}/{NN}-{persona.slug}.md using this template:

# Review by {name}
**Persona:** {title} | **Archetype:** {archetype} | **Date:** {ISO date}
**Target:** {target}

## First Impressions
{2-3 paragraph reaction from your perspective}

## Detailed Analysis
### {Priority Area 1}
{Analysis with evidence} — **Rating:** {1-5}/5
### {Priority Area 2}
{Analysis with evidence} — **Rating:** {1-5}/5
### {Priority Area 3}
{Analysis with evidence} — **Rating:** {1-5}/5

## Strengths
1-3 specific strengths with evidence

## Concerns
1-3 specific concerns with evidence and impact

## Recommendations
1-3 actionable recommendations with **Priority: HIGH/MEDIUM/LOW**

## Overall Rating
**Score:** {1-10}/10 | **Summary:** {one sentence} | **Would recommend:** {Yes/Yes with caveats/No}

After writing, read the file back to verify.
")
```

Wait for each batch to complete before starting the next.
Announce: `[FOCUS GROUP] Batch {B} complete ({N}/{batch_size} succeeded)`

**Sequential mode:** Run each persona one at a time. Announce: `[FOCUS GROUP] Review {NN}/{count} complete: {persona.name}`

---

### Phase 5.5: Quality Validation

**Skip if `--no-quality-check` is set.**

For each completed review file:
```
Agent(subagent_type="focusgroup:quality-enforcer", model="sonnet", prompt="
Validate this focus group review:
REVIEW TEXT: {read the review file content}
PROBE DATA: {probe-data.json content if available, otherwise 'N/A'}
TARGET DESCRIPTION: {target}
")
```

Parse quality score. **Threshold:** 60 (or 75 if `--strict`).

If score < threshold: re-spawn reviewer with feedback: "QUALITY FEEDBACK: Your review scored {score}/100. Fix: {issues}." Max 1 retry per reviewer. If retry fails, accept with warning.

Track quality scores for meta.json.
Announce: `[FOCUS GROUP] Quality: {passed}/{total} passed (threshold: {threshold})`

---

### Phase 6: Write Metadata

Write `{out_dir}/meta.json`:
```json
{
  "target": "{target}", "input_type": "{type}", "panel": "{panel}",
  "mode": "{mode}", "count": {count}, "date": "{ISO date}",
  "methodology": "{methodology or 'standard'}", "deep": true/false,
  "personas": [{"number": 1, "name": "{name}", "slug": "{slug}", "archetype": "{archetype}", "review_file": "{NN}-{slug}.md", "completed": true/false}],
  "completed_reviews": {N}, "failed_reviews": [{failed numbers}],
  "quality_scores": {"{persona_slug}": {score}},
  "outputs_generated": ["action-plan.md", "swot.md", "risk-impact-matrix.md", "report.html"]
}
```

---

### Phase 7: Synthesize

Read all completed review files from `{out_dir}/`. Spawn ONE opus agent:

```
Agent(subagent_type="focusgroup:focus-director", model="opus", prompt="
[SYNTHESIS]
You are the Focus Group Director synthesizing results of a multi-persona review.
TARGET: {target} | PANEL: {panel} | COMPLETED: {N} of {count}
{If any failed: 'FAILED: {list} — note gaps in synthesis.'}

Read ALL review files in {out_dir}/ (##-*.md, NOT synthesis.md/meta.json).
Write synthesis to {out_dir}/synthesis.md with these sections:

# Focus Group Synthesis
**Target:** {target} | **Date:** {date} | **Panel:** {panel} | **Personas:** {N}/{count}

## Executive Summary — 3-4 paragraph synthesis of all findings
## Consensus Findings
### Unanimous Agreements — things all/nearly all agreed on with quotes
### Majority Views — things 6+ agreed on, noting dissent
## Points of Contention — for each disagreement: For/Against/Implication
## Aggregate Ratings — table: Persona|Archetype|Score|Recommend, plus mean/median/range/stddev
## Top Recommendations (by frequency) — table: #|Recommendation|Priority|Mentioned By|Count
## Strengths Heat Map — table: Strength|Personas|Frequency
## Risk Assessment — table: Risk|Severity|Personas|Frequency
## Blind Spots — uncovered areas, missing perspectives, unanswered questions
## Methodology — panel type, mode, counts, target type, note on dynamic generation

Write using Write tool. Read back to verify.
")
```

---

### Phase 7.5: Generate Outputs

**Skip if no output flags are set.** If `--full-report`, enable all four outputs.

**Step 1 (if action_plan):** Generate action plan FIRST (required by risk-matrix):
```
Agent(subagent_type="focusgroup:action-plan-generator", model="sonnet", prompt="
Read synthesis and reviews from {out_dir}/. Format: {backlog_format}. Target: {target}. Date: {date}.
Write to {out_dir}/action-plan.{md|csv} or {out_dir}/issues/ for github format.
")
```

**Step 2 (parallel):** Spawn remaining outputs simultaneously with `run_in_background=true`:

- {If swot:} `focusgroup:swot-generator` model="haiku" — Read `{out_dir}/synthesis.md`, write `{out_dir}/swot.md`
- {If risk_matrix AND action_plan done:} `focusgroup:risk-impact-generator` model="haiku" — Read `{out_dir}/action-plan.md`, write `{out_dir}/risk-impact-matrix.md`
- {If html_report:} `focusgroup:html-report-generator` model="sonnet" — Read all `{out_dir}/` files, write `{out_dir}/report.html`

Wait for all to complete. Announce: `[FOCUS GROUP] Generated: {list of output files}`

---

### Phase 8: JSON Output (if --format json)

If `--format json`, write `{out_dir}/report.json`:
```json
{
  "meta": {meta.json contents},
  "synthesis": {
    "average_score": N, "median_score": N, "score_range": [min, max],
    "consensus": [...], "contentions": [...],
    "top_recommendations": [{"text": "...", "priority": "HIGH", "count": N}],
    "top_strengths": [{"text": "...", "count": N}],
    "top_risks": [{"text": "...", "severity": "HIGH", "count": N}]
  },
  "reviews": [{"persona": {...}, "scores": {...}, "would_recommend": "...", "strengths": [...], "concerns": [...], "recommendations": [...]}]
}
```

Parse from the markdown review files and synthesis.

---

### Phase 9: Report

```
[FOCUS GROUP COMPLETE]

Target: {target}
Panel: {panel}
Reviews: {N}/{count} completed
Output: {out_dir}/
{If methodology: Methodology: {methodology name}}
{If deep: Protocol: Deep review (full protocol phases)}
{If quality scores: Quality: {avg_score}/100 average}

Average Score: {score}/10
Recommendation: {majority recommendation}

Top Consensus Findings:
1-3 findings

Top Recommendations:
1-3 recommendations (mentioned by {N} reviewers each)

Generated Outputs:
{list each generated output file, if any}

Files:
{list each file in out_dir/}

Read {out_dir}/synthesis.md for the full analysis.
```

---

## Error Recovery

| Error | Action |
|-------|--------|
| Persona generation returns fewer than requested | Continue with available personas, note in synthesis |
| Individual reviewer agent fails | Skip, continue with others, note gap in synthesis |
| Synthesis agent fails | Retry once; if still fails, output raw review list |
| Output directory write fails | Try `.focusgroup/` without timestamp, inform user |
| Browser tools unavailable for website | Fall back to WebFetch; note limited analysis |
| Custom personas file not found/malformed | Warn user, fall back to auto-generation |
| Methodology protocol file missing | Fall back to standard single-round review; warn user |
| Quality enforcer fails | Skip quality check for that review; note in meta.json |
| Output generator fails | Skip that output; list what was generated in Phase 9 |
| Website probe fails | Continue without probe data; reviewers use direct browsing |
| --compare with no competitor URLs | Error: "--compare requires at least 1 competitor target" |

## Cancellation

User can cancel at any time. In-progress reviews will be saved. Partial results remain in the output directory.