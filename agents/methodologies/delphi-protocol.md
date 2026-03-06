# Delphi Method Protocol

This protocol defines a 3-round iterative review process where reviewers refine their positions based on anonymized group feedback. The command reads this document and follows each round's instructions sequentially.

**IMPORTANT:** This document is an instruction set for the command orchestrator. It does NOT spawn agents. The command spawns all agents directly.

---

## Overview

| Property | Value |
|----------|-------|
| Rounds | 3 |
| Output dirs | `{out_dir}/round-1/`, `round-2/`, `round-3/` |
| Final outputs | `{out_dir}/delphi-evolution.md`, `{out_dir}/synthesis.md` |
| Time factor | ~3x standard review |

---

## Round 1: Blind Review

### Setup
```bash
mkdir -p {out_dir}/round-1
```

### Reviewer Spawn Prompt Template
Spawn each reviewer with the standard review prompt (same as non-methodology mode). Each reviewer writes to `{out_dir}/round-1/{NN}-{slug}.md`.

No changes to the standard reviewer prompt for Round 1 — this is a baseline blind review.

### After Round 1
Read all completed reviews from `{out_dir}/round-1/`. Count: {completed_count} of {total_count}.

---

## Inter-Round Aggregation (1 → 2)

The command (NOT an agent) performs this aggregation:

1. **Read all Round 1 reviews** from `{out_dir}/round-1/`
2. **Extract findings**: Pull all items from Strengths, Concerns, and Recommendations sections
3. **Anonymize**: Replace persona names with "Reviewer A", "Reviewer B", etc. (alphabetical by file order)
4. **Calculate agreement**: For each finding, count how many reviewers mentioned it or a similar point
5. **Produce anonymized summary** with this format:

```
ROUND 1 ANONYMIZED SUMMARY
===========================

FINDINGS BY AGREEMENT LEVEL:

UNANIMOUS (mentioned by all reviewers):
- {finding} — Reviewers: {A, B, C, ...}

MAJORITY (mentioned by 50%+ of reviewers):
- {finding} — Reviewers: {A, C, E} ({N}/{total})

MINORITY (mentioned by <50% of reviewers):
- {finding} — Reviewers: {B} ({N}/{total})

UNIQUE (mentioned by only 1 reviewer):
- {finding} — Reviewer: {D}

RATING DISTRIBUTION:
- Average overall score: {mean}/10
- Score range: {min}-{max}/10
- Standard deviation: {std_dev}
- Individual scores (anonymized): A={score}, B={score}, ...

AREAS OF DISAGREEMENT:
- {topic}: Reviewer {A} rated {X}/5, Reviewer {D} rated {Y}/5 (delta: {diff})
```

Write this summary to `{out_dir}/round-1-summary.md`.

---

## Round 2: Informed Revision

### Setup
```bash
mkdir -p {out_dir}/round-2
```

### Reviewer Spawn Prompt Template
Spawn each reviewer with this modified prompt (append to standard review prompt):

```
DELPHI ROUND 2 — INFORMED REVISION

You previously reviewed this target in Round 1. Your Round 1 review is below.
You have also received an anonymized summary of ALL Round 1 reviews.

YOUR ROUND 1 REVIEW:
{paste the reviewer's own Round 1 review text}

ANONYMIZED GROUP SUMMARY:
{paste the round-1-summary.md content}

INSTRUCTIONS FOR ROUND 2:
1. Re-read your Round 1 review and the group summary
2. Consider points raised by other reviewers that you missed
3. Revise your ratings if the group evidence changed your assessment
4. For EACH rating change, explicitly state: "Changed from X to Y because: {reason}"
5. Add new findings you now agree with from other reviewers
6. Remove or downgrade findings you no longer believe are valid
7. Keep your persona perspective — don't just agree with everyone

Write your revised review to: {out_dir}/round-2/{NN}-{slug}.md

Use the same template as Round 1, but add a "Revisions" section at the end:

## Revisions from Round 1
- {What changed and why, for each change}
- {If nothing changed: "I maintain all my original positions because: {reasons}"}
```

### After Round 2
Read all Round 2 reviews. Track which reviewers changed ratings and by how much.

---

## Inter-Round Aggregation (2 → 3)

The command performs convergence analysis:

1. **Compare Round 1 and Round 2 ratings** for each reviewer
2. **Calculate convergence delta**: How much did the score spread narrow?
3. **Identify shifts**: Which reviewers changed positions? On what topics?
4. **Produce convergence report**:

```
CONVERGENCE ANALYSIS (Round 1 → Round 2)
=========================================

SCORE CONVERGENCE:
- Round 1 spread: {min}-{max} (range: {range1}, std dev: {std1})
- Round 2 spread: {min}-{max} (range: {range2}, std dev: {std2})
- Convergence: {decreased/increased/stable} by {delta}

OPINION SHIFTS:
- Reviewer {A}: Score {old} → {new} ({+/-change}). Reason: "{quoted reason}"
- Reviewer {C}: Score {old} → {new} ({+/-change}). Reason: "{quoted reason}"

FINDINGS THAT GAINED SUPPORT:
- {finding}: Round 1: {N} reviewers → Round 2: {M} reviewers

FINDINGS THAT LOST SUPPORT:
- {finding}: Round 1: {N} reviewers → Round 2: {M} reviewers

PERSISTENT DISAGREEMENTS:
- {topic}: {Reviewer A} maintains {position} vs {Reviewer D} maintains {position}
```

Write to `{out_dir}/round-2-convergence.md`.

---

## Round 3: Final Positions

### Setup
```bash
mkdir -p {out_dir}/round-3
```

### Reviewer Spawn Prompt Template

```
DELPHI ROUND 3 — FINAL POSITION

This is the final round. State your definitive assessment.

YOUR ROUND 2 REVIEW:
{paste the reviewer's Round 2 review text}

CONVERGENCE ANALYSIS:
{paste the round-2-convergence.md content}

INSTRUCTIONS FOR ROUND 3:
1. This is your FINAL position — make it count
2. For each finding, state one of:
   - "I MAINTAIN: {finding} because {evidence-based reason}"
   - "I CHANGED: From {old position} to {new position} because {evidence-based reason}"
   - "I WITHDRAW: {finding} because {reason}"
3. Provide your final overall score with full justification
4. If you disagree with the group majority, explicitly state why and stand your ground

Write your final review to: {out_dir}/round-3/{NN}-{slug}.md
```

---

## Evolution Trace

After Round 3, the command generates `{out_dir}/delphi-evolution.md`:

```markdown
# Delphi Evolution Trace

## Score Evolution

| Persona | Round 1 | Round 2 | Round 3 | Total Shift |
|---------|---------|---------|---------|-------------|
| {name} | {score} | {score} | {score} | {+/-change} |

## Convergence Metrics
- Round 1 std dev: {val}
- Round 2 std dev: {val}
- Round 3 std dev: {val}
- Final consensus level: {High/Medium/Low} (std dev < 1.0 = High, < 2.0 = Medium, >= 2.0 = Low)

## Finding Evolution
{For each major finding, track support across rounds}

## Key Shifts
{Notable opinion changes with quoted reasons}
```

---

## Synthesis Enhancement

When spawning the synthesis agent (focus-director), append to the standard synthesis prompt:

```
METHODOLOGY: Delphi (3-round iterative)

Additional context for synthesis:
- Read the evolution trace at {out_dir}/delphi-evolution.md
- Read all three rounds of reviews
- Weight Round 3 positions most heavily (they are the most informed)
- Note the convergence level in your executive summary
- Highlight findings where the group FAILED to converge — these are genuine points of contention
- Include the score evolution table in your Aggregate Ratings section
```
