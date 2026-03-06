# Debate Method Protocol

This protocol defines a structured Red Team vs Blue Team debate with a Judge persona. The command reads this document and follows each round's instructions.

**IMPORTANT:** This document is an instruction set for the command orchestrator. It does NOT spawn agents. The command spawns all agents directly.

---

## Overview

| Property | Value |
|----------|-------|
| Rounds | 3 (Opening, Rebuttal, Closing) + Judge |
| Teams | Red (critics/attackers) + Blue (defenders/advocates) |
| Output dirs | `{out_dir}/red-team/`, `{out_dir}/blue-team/` |
| Final outputs | `{out_dir}/judge-scores.md`, `{out_dir}/debate-report.md`, `{out_dir}/synthesis.md` |

---

## Setup: Team Assignment

Split personas into two teams based on archetype characteristics:

**Red Team (Critics/Attackers)** — personas with skeptical, analytical, security-focused, or quality-focused archetypes:
- Keywords: skeptic, critic, security, quality, performance, compliance, risk, auditor, tester
- Role: Find vulnerabilities, weaknesses, risks, and problems

**Blue Team (Defenders/Advocates)** — personas with constructive, creative, user-focused, or business-focused archetypes:
- Keywords: advocate, designer, user, business, product, growth, innovation, accessibility, content
- Role: Present strengths, defend design decisions, highlight value

**Team sizes:** Split as evenly as possible. If odd number, assign the extra persona to Red Team (critics need more voices for balance). Minimum 2 per team.

**Judge persona:** Generate ONE additional persona — a senior industry expert with balanced perspective. The Judge does NOT participate in debate rounds. The Judge evaluates after all rounds complete.

```bash
mkdir -p {out_dir}/red-team {out_dir}/blue-team
```

---

## Round 1: Opening Arguments

### Red Team Spawn Prompt Template

```
DEBATE MODE — RED TEAM OPENING ARGUMENT

You are on the RED TEAM (critics/attackers). Your job is to find and present the strongest possible case AGAINST this target.

TARGET: {target}
YOUR PERSONA: {persona details}

INSTRUCTIONS:
1. Use your tools to thoroughly examine the target
2. Find vulnerabilities, weaknesses, risks, poor decisions, missing features
3. Present your TOP 5 criticisms, ranked by severity
4. Each criticism MUST include:
   - Specific evidence (URL, file path, line number, metric)
   - Severity rating: Critical / Major / Minor
   - Real-world impact: What goes wrong because of this?
5. Be aggressive but fair — every claim must have evidence
6. Do NOT acknowledge strengths — that's Blue Team's job

Write to: {out_dir}/red-team/round-1-{NN}-{slug}.md

Format:
# Red Team Opening — {name}

## Criticism 1: {title}
**Severity:** {Critical/Major/Minor}
**Evidence:** {specific reference}
**Impact:** {what goes wrong}
**Details:** {2-3 paragraphs}

[repeat for criticisms 2-5]

## Summary Prosecution
{1 paragraph: strongest overall case against the target}
```

### Blue Team Spawn Prompt Template

```
DEBATE MODE — BLUE TEAM OPENING ARGUMENT

You are on the BLUE TEAM (defenders/advocates). Your job is to present the strongest possible case FOR this target.

TARGET: {target}
YOUR PERSONA: {persona details}

INSTRUCTIONS:
1. Use your tools to thoroughly examine the target
2. Find strengths, smart decisions, good patterns, competitive advantages
3. Present your TOP 5 strengths, ranked by importance
4. Each strength MUST include:
   - Specific evidence (URL, file path, line number, metric)
   - Importance: Strategic / Tactical / Quality-of-Life
   - Business value: Why does this matter?
5. Be enthusiastic but honest — every claim must have evidence
6. Do NOT acknowledge weaknesses — that's Red Team's job

Write to: {out_dir}/blue-team/round-1-{NN}-{slug}.md

Format:
# Blue Team Opening — {name}

## Strength 1: {title}
**Importance:** {Strategic/Tactical/Quality-of-Life}
**Evidence:** {specific reference}
**Business Value:** {why it matters}
**Details:** {2-3 paragraphs}

[repeat for strengths 2-5]

## Summary Defense
{1 paragraph: strongest overall case for the target}
```

Spawn Red Team and Blue Team personas in parallel (all with `run_in_background: true`).

---

## Round 2: Rebuttals

After Round 1 completes, read all Red Team and Blue Team opening arguments.

### Red Team Rebuttal Prompt Template

```
DEBATE MODE — RED TEAM REBUTTAL

BLUE TEAM'S OPENING ARGUMENTS:
{paste ALL Blue Team Round 1 content}

YOUR ROUND 1 ARGUMENT:
{paste this reviewer's Round 1 content}

INSTRUCTIONS:
1. Address Blue Team's strongest points — can you undermine their evidence?
2. Present counter-evidence that weakens their claims
3. Reinforce your strongest criticisms that Blue Team failed to address
4. Add NEW criticisms discovered while reviewing Blue Team's evidence
5. Every counter-argument must cite specific evidence

Write to: {out_dir}/red-team/round-2-{NN}-{slug}.md
```

### Blue Team Rebuttal Prompt Template

```
DEBATE MODE — BLUE TEAM REBUTTAL

RED TEAM'S OPENING ARGUMENTS:
{paste ALL Red Team Round 1 content}

YOUR ROUND 1 ARGUMENT:
{paste this reviewer's Round 1 content}

INSTRUCTIONS:
1. Address Red Team's criticisms — can you defend against them?
2. Present evidence that mitigates or refutes their concerns
3. Acknowledge valid criticisms honestly (concede when evidence is strong)
4. Reinforce your strongest points that Red Team failed to counter
5. Every defense must cite specific evidence

Write to: {out_dir}/blue-team/round-2-{NN}-{slug}.md
```

---

## Round 3: Closing Arguments

### Both Teams Closing Prompt Template

```
DEBATE MODE — {RED/BLUE} TEAM CLOSING ARGUMENT

FULL DEBATE HISTORY:
{paste all Round 1 and Round 2 content from BOTH teams}

INSTRUCTIONS:
1. Present your 3 STRONGEST remaining points after the full debate
2. For each point, explain why it survived the opposing team's challenges
3. Concede any points where the opposing team had stronger evidence
4. State your final verdict: Is this target {strong/acceptable/weak/unacceptable}?

Write to: {out_dir}/{red-team|blue-team}/round-3-{NN}-{slug}.md
```

---

## Judge Evaluation

After all 3 rounds, spawn the Judge persona:

```
DEBATE MODE — JUDGE EVALUATION

You are an impartial judge evaluating a structured Red Team vs Blue Team debate.

Read ALL debate files in {out_dir}/red-team/ and {out_dir}/blue-team/.

For EACH exchange topic, score on these criteria (1-10 each):
- Evidence Quality: How well-supported were the claims?
- Argument Strength: How logically sound was the reasoning?
- Practical Impact: How much does this matter in the real world?

Determine a winner for each exchange and an overall winner.

Write to: {out_dir}/judge-scores.md

Format:
# Judge Evaluation

## Exchange 1: {topic}
| Criterion | Red Team | Blue Team |
|-----------|----------|-----------|
| Evidence Quality | {1-10} | {1-10} |
| Argument Strength | {1-10} | {1-10} |
| Practical Impact | {1-10} | {1-10} |
| **Total** | **{sum}** | **{sum}** |
**Winner:** {Red/Blue Team}
**Reasoning:** {1-2 sentences}

[repeat for each exchange]

## Overall Verdict
**Winner:** {Red/Blue Team} by {margin}
**Key Factor:** {what decided it}
```

---

## Vulnerability Report

After Judge evaluation, the command generates `{out_dir}/debate-report.md`:

```markdown
# Debate Vulnerability Report

## Defense Strength Ratings

| Finding | Red Team Evidence | Blue Team Defense | Rating |
|---------|-------------------|-------------------|--------|
| {finding} | {summary} | {summary} | {Undefended/Weakly Defended/Strongly Defended/Refuted} |

### Rating Definitions
- **Undefended:** Red Team raised it, Blue Team did not address it
- **Weakly Defended:** Blue Team responded but with weaker evidence
- **Strongly Defended:** Blue Team provided compelling counter-evidence
- **Refuted:** Blue Team demonstrated the criticism is invalid

## Priority Actions
{Items rated "Undefended" or "Weakly Defended" — these need immediate attention}
```

---

## Synthesis Enhancement

When spawning the synthesis agent, append:

```
METHODOLOGY: Debate (Red Team vs Blue Team, 3 rounds + Judge)

Additional context:
- Read the judge scores at {out_dir}/judge-scores.md
- Read the vulnerability report at {out_dir}/debate-report.md
- Emphasize findings rated "Undefended" — these are confirmed weaknesses
- Note findings rated "Refuted" — initial concerns that were disproven
- The debate format provides higher confidence in findings that survived adversarial challenge
- Include judge's overall verdict in executive summary
```
