# Cognitive Walkthrough Protocol

This protocol defines a task-completion walkthrough where each persona narrates their attempt to achieve a specific user goal. The command reads this document and follows the instructions.

**IMPORTANT:** This document is an instruction set for the command orchestrator. It does NOT spawn agents. The command spawns all agents directly.

---

## Overview

| Property | Value |
|----------|-------|
| Input flag | `--walkthrough "user goal"` |
| Steps | 5-15 discrete steps decomposed from the goal |
| Output dirs | `{out_dir}/narratives/` |
| Final outputs | `{out_dir}/walkthrough-steps.md`, `{out_dir}/friction-map.md`, `{out_dir}/synthesis.md` |

---

## Step 1: Goal Decomposition

The command spawns the focus-director to decompose the user goal into discrete steps:

### Director Spawn Prompt for Decomposition

```
WALKTHROUGH GOAL DECOMPOSITION

USER GOAL: "{walkthrough_goal}"
TARGET: {target}

Examine the target and decompose this goal into 5-15 discrete, sequential steps a user would take.

For websites: Navigate the site and identify each page/action in the flow.
For codebases: Identify each file/function/command a developer would need.
For ideas: Identify each decision/action needed to validate or implement.

Output format (write to {out_dir}/walkthrough-steps.md):

# Walkthrough Steps

**Goal:** {walkthrough_goal}
**Target:** {target}
**Steps:** {N}

## Step 1: {action verb phrase}
**Page/Location:** {where this happens}
**Expected Action:** {what the user does}
**Success Indicator:** {how to know it worked}

## Step 2: {action verb phrase}
...

## Step {N}: {action verb phrase — goal achieved}
**Success Indicator:** Goal is complete when {measurable outcome}
```

After the director completes, read `{out_dir}/walkthrough-steps.md` and parse the steps.

---

## Step 2: Persona Walkthroughs

### Setup
```bash
mkdir -p {out_dir}/narratives
```

### Reviewer Spawn Prompt Template

Spawn each reviewer with this prompt (replaces standard review prompt):

```
COGNITIVE WALKTHROUGH MODE

You are narrating your attempt to complete a specific goal on this target, step by step.

YOUR PERSONA: {persona details}
TARGET: {target}
GOAL: "{walkthrough_goal}"

STEPS TO COMPLETE:
{paste walkthrough-steps.md content}

INSTRUCTIONS:
For EACH step, narrate your experience as {persona.name}:

1. **What I see:** Describe what's visible/available at this point
2. **What I try:** What action do you take? (Actually use your tools to attempt it)
3. **What happens:** What's the result?
4. **Confusion points:** Anything unclear, misleading, or frustrating?
5. **Friction score:** Rate 1-5:
   - 1 = Effortless, immediately obvious
   - 2 = Minor hesitation but figured it out
   - 3 = Required some thought or exploration
   - 4 = Significant confusion or difficulty
   - 5 = Blocked or abandoned — could not complete
6. **Drop-off probability:** Based on your persona's patience and expertise, would a real user like you give up at this point? (0-100%)

IMPORTANT:
- ACTUALLY use your tools to attempt each step (navigate, click, read, etc.)
- Your persona's expertise level affects how you experience each step
- A developer persona finds CLI steps easy; a non-technical persona finds them hard
- Be honest about confusion — don't pretend steps are easy if they aren't
- If you cannot complete a step, narrate WHY and score it 5

Write to: {out_dir}/narratives/{slug}.md

Format:
# Walkthrough by {name}
**Persona:** {title}
**Goal:** {walkthrough_goal}
**Expertise Level:** {novice/intermediate/expert in this domain}

## Step 1: {step title}
**What I see:** {description}
**What I try:** {action taken}
**What happens:** {result}
**Confusion:** {any confusion or none}
**Friction:** {1-5}/5
**Drop-off risk:** {0-100}%

[repeat for each step]

## Journey Summary
**Steps completed:** {N} of {total}
**Steps abandoned:** {list any scored 5}
**Average friction:** {mean}/5
**Cumulative drop-off:** {probability of reaching the end}
**Overall verdict:** {Would I achieve this goal? Yes/Partially/No}
**Biggest friction point:** {which step and why}
```

---

## Step 3: Friction Map Generation

After all walkthrough narratives complete, the command generates `{out_dir}/friction-map.md`:

```markdown
# Friction Map

**Goal:** {walkthrough_goal}
**Personas:** {N} completed walkthroughs

## Friction Heatmap

| Step | Description | {Persona1} | {Persona2} | ... | Average | Max |
|------|-------------|------------|------------|-----|---------|-----|
| 1 | {title} | {score} | {score} | ... | {avg} | {max} |
| 2 | {title} | {score} | {score} | ... | {avg} | {max} |
...

## Drop-off Funnel

| Step | Cumulative Survival Rate | Drop-off This Step |
|------|--------------------------|-------------------|
| Start | 100% | - |
| Step 1 | {100 - avg_dropout_1}% | {avg_dropout_1}% |
| Step 2 | {cumulative}% | {avg_dropout_2}% |
...
| Final | {completion_rate}% | - |

**Estimated completion rate:** {final survival}%

## Critical Friction Points (Average >= 3.5)

### {Step N}: {title}
- **Average friction:** {score}/5
- **Personas who struggled:** {names and their scores}
- **Common complaint:** {most frequent confusion point}
- **Recommendation:** {what to fix}

## Accessibility Concerns
{Steps where non-expert personas scored significantly higher friction than expert personas}

## Persona Completion Summary

| Persona | Steps Completed | Average Friction | Would Complete Goal? |
|---------|-----------------|-----------------|---------------------|
| {name} | {N}/{total} | {avg}/5 | {Yes/Partially/No} |
```

The command calculates these values by reading all narrative files and extracting friction scores and drop-off probabilities.

---

## Synthesis Enhancement

When spawning the synthesis agent, append:

```
METHODOLOGY: Cognitive Walkthrough
GOAL: "{walkthrough_goal}"

Additional context:
- Read the friction map at {out_dir}/friction-map.md
- Read all walkthrough narratives in {out_dir}/narratives/
- Focus synthesis on the friction funnel — where do users drop off?
- Highlight steps where expert and novice personas diverged significantly
- Recommendations should prioritize fixing highest-friction steps first
- Include the estimated completion rate prominently in executive summary
```
