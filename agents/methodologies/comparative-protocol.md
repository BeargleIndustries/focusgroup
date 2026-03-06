# Comparative Review Protocol

This protocol defines a multi-target comparative review where each persona reviews all targets on identical criteria. The command reads this document and follows the instructions.

**IMPORTANT:** This document is an instruction set for the command orchestrator. It does NOT spawn agents. The command spawns all agents directly.

---

## Overview

| Property | Value |
|----------|-------|
| Input flag | `--compare target2 [target3] [target4]` |
| Max targets | 4 (primary + up to 3 competitors) |
| Output dirs | `{out_dir}/reviews/{slug}/` per persona |
| Final outputs | `{out_dir}/competitive-matrix.md`, `{out_dir}/gap-analysis.md`, `{out_dir}/synthesis.md` |

---

## Input Parsing

The `--compare` flag consumes the next 1-3 non-flag arguments as competitor targets.

```
Primary target: {target} (from main argument)
Competitors: {compare_targets} (from --compare args)
All targets: [{target}, {compare_targets...}]
```

If more than 3 competitor targets provided:
- Warn: "Comparative review limited to 4 total targets. Using first 3 competitors."
- Truncate to 3 competitors

Generate a slug for each target (domain name for URLs, directory name for paths, first 3 words for ideas).

---

## Review Execution

### Setup
```bash
for each persona_slug:
  mkdir -p {out_dir}/reviews/{persona_slug}
```

### Reviewer Spawn Prompt Template

Each reviewer reviews ALL targets (primary + competitors) in a single session:

```
COMPARATIVE REVIEW MODE

You are reviewing multiple targets on IDENTICAL criteria to enable fair comparison.

YOUR PERSONA: {persona details}
INPUT TYPE: {input_type}

TARGETS TO REVIEW:
1. [PRIMARY] {target} (slug: {target_slug})
2. [COMPETITOR] {compare_target_1} (slug: {comp_slug_1})
3. [COMPETITOR] {compare_target_2} (slug: {comp_slug_2})
{if applicable: 4. [COMPETITOR] {compare_target_3} (slug: {comp_slug_3})}

{protocol_loading_instructions if --deep is also set}

INSTRUCTIONS:
1. Review EACH target using the same criteria from your review priorities
2. Use your tools to examine each target individually
3. For EACH target, write a separate review file
4. After reviewing all targets, write a comparison file

IMPORTANT:
- Apply the SAME evaluation criteria to every target
- Rate each target on your 3 priority areas using the same rubric
- Note features present in one target but missing in others
- Be specific about WHERE each target is better or worse

Write individual reviews to:
- {out_dir}/reviews/{persona_slug}/{target_slug}.md
- {out_dir}/reviews/{persona_slug}/{comp_slug_1}.md
- {out_dir}/reviews/{persona_slug}/{comp_slug_2}.md

Write your comparison to:
- {out_dir}/reviews/{persona_slug}/comparison.md

Individual review format: (same as standard review template)

Comparison format:
# Comparative Assessment by {name}

## Side-by-Side Ratings

| Dimension | {target} | {comp1} | {comp2} | Winner |
|-----------|----------|---------|---------|--------|
| {priority_1} | {1-5} | {1-5} | {1-5} | {name} |
| {priority_2} | {1-5} | {1-5} | {1-5} | {name} |
| {priority_3} | {1-5} | {1-5} | {1-5} | {name} |
| **Overall** | **{1-10}** | **{1-10}** | **{1-10}** | **{name}** |

## Where {primary target} Leads
{Specific areas where the primary target is superior, with evidence}

## Where {primary target} Trails
{Specific areas where competitors are superior, with evidence}

## Feature Gaps
| Feature | {target} | {comp1} | {comp2} |
|---------|----------|---------|---------|
| {feature} | ✅/❌/Partial | ✅/❌/Partial | ✅/❌/Partial |

## Recommendation
{Which would this persona choose and why?}
```

---

## Competitive Matrix Generation

After all reviews complete, the command generates `{out_dir}/competitive-matrix.md`:

```markdown
# Competitive Matrix

**Primary target:** {target}
**Competitors:** {list}
**Reviewers:** {count}

## Aggregate Scores

| Target | Average Score | Median | Min | Max | Recommendation Rate |
|--------|--------------|--------|-----|-----|-------------------|
| {target} | {avg}/10 | {med} | {min} | {max} | {N}% would recommend |
| {comp1} | {avg}/10 | {med} | {min} | {max} | {N}% would recommend |

## Dimension-by-Dimension Comparison

| Dimension | {target} Avg | {comp1} Avg | {comp2} Avg | Leader |
|-----------|-------------|-------------|-------------|--------|
| {dim1} | {avg}/5 | {avg}/5 | {avg}/5 | {name} |

## Positioning

| Position | Target | Evidence |
|----------|--------|----------|
| **Clear Leader** | {target if leading in 60%+ dimensions} | {summary} |
| **Competitive** | {targets within 0.5 of leader} | {summary} |
| **Behind** | {targets trailing in 60%+ dimensions} | {summary} |

## Feature Presence Matrix

| Feature | {target} | {comp1} | {comp2} |
|---------|----------|---------|---------|
{aggregate feature presence from all reviewer comparisons}
```

---

## Gap Analysis Generation

The command generates `{out_dir}/gap-analysis.md`:

```markdown
# Gap Analysis: {target} vs Competitors

## Where {target} Leads (Competitive Advantages)
{Dimensions/features where {target} scores highest, with evidence frequency}

## Where {target} Trails (Competitive Gaps)
{Dimensions/features where competitors score higher}
| Gap | Leader | {target} Score | Leader Score | Delta | Priority |
|-----|--------|---------------|-------------|-------|----------|

## Parity Areas
{Dimensions where all targets score similarly — not a differentiator}

## Strategic Recommendations
1. **Protect:** {advantages to maintain}
2. **Close:** {gaps to close, prioritized by impact}
3. **Differentiate:** {unique opportunities no competitor addresses}
```

---

## Synthesis Enhancement

When spawning the synthesis agent, append:

```
METHODOLOGY: Comparative Review
TARGETS: {target} (primary) vs {competitors}

Additional context:
- Read the competitive matrix at {out_dir}/competitive-matrix.md
- Read the gap analysis at {out_dir}/gap-analysis.md
- Read individual comparison files from each persona
- Frame synthesis around competitive positioning, not absolute quality
- Identify clear "winner" per dimension with supporting evidence
- Recommendations should focus on closing competitive gaps
```
