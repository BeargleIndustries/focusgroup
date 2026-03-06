# Confidence Scoring Protocol

This protocol defines how findings are confidence-scored during synthesis. The focus-director reads this document and applies the scoring formula to each finding.

---

## Scoring Formula

Each finding receives a confidence score calculated as:

```
base_confidence = independent_agreement_rate
adjustments = adversarial_survival + evidence_specificity + diversity_factor
final_score = clamp(base_confidence + adjustments, 0.0, 1.0)
```

### Component 1: Independent Agreement Rate (Base)

```
base_confidence = N / M
```

Where:
- N = number of reviewers who independently mentioned this finding (or a substantially similar one)
- M = total number of completed reviewers

"Substantially similar" means the same core observation, even if phrased differently. The synthesizer must use judgment to group related findings.

### Component 2: Adversarial Survival (Debate mode only)

| Condition | Adjustment |
|-----------|------------|
| Finding survived Red Team challenge with strong evidence | +0.20 |
| Finding survived Red Team challenge with weak evidence | +0.10 |
| Finding was conceded by opposing team | +0.15 |
| Not applicable (non-Debate mode) | +0.00 |

### Component 3: Evidence Specificity

| Evidence Type | Adjustment |
|---------------|------------|
| Backed by quantified metrics (numbers, measurements) | +0.10 |
| Backed by direct quotes from target | +0.05 |
| Backed by tool output (probe data, command results) | +0.10 |
| Backed by external sources (URLs, research) | +0.05 |
| No specific evidence cited | -0.20 |

Multiple evidence types stack, but cap the total evidence adjustment at +0.15.

### Component 4: Diversity Factor

```
diversity_bonus = (distinct_archetypes - 1) * 0.10
```

Where:
- distinct_archetypes = number of different reviewer archetypes that mentioned this finding
- Cap at +0.30 (4+ distinct archetypes)

A finding raised by a security expert, a designer, and a business analyst (3 archetypes) is more robust than one raised by 3 security experts.

---

## Confidence Tiers

| Tier | Score Range | Display | Interpretation |
|------|-----------|---------|----------------|
| Very High | > 0.85 | 🟢 VH ({score}) | Near-universal agreement with strong evidence |
| High | 0.70 - 0.85 | 🟢 H ({score}) | Majority agreement with good evidence |
| Medium | 0.50 - 0.70 | 🟡 M ({score}) | Moderate agreement or mixed evidence |
| Low | 0.30 - 0.50 | 🟠 L ({score}) | Minority view or weak evidence |
| Very Low | < 0.30 | 🔴 VL ({score}) | Single reviewer or no supporting evidence |

---

## Display Format

In the synthesis report, tag each finding with its confidence:

```
- **Finding text here** — Confidence: 🟢 H (0.78) [4/7 reviewers, 3 archetypes, metric-backed]
```

The bracketed details show: [agreement rate, archetype count, evidence type].

---

## Aggregation Rules

When multiple reviewers raise the same finding with different levels of specificity:
1. Use the MOST specific version of the finding
2. Credit ALL reviewers who raised it (even if their version was less specific)
3. Use the STRONGEST evidence cited by any reviewer

When findings partially overlap:
1. If >70% overlap in meaning, merge into one finding
2. If <70% overlap, keep as separate findings
3. Note the relationship: "Related to: {other finding}"

---

## Special Cases

### Delphi Mode
- Use Round 3 (final) positions for agreement rate
- Note findings that GAINED confidence across rounds (convergence bonus: +0.05)
- Note findings that LOST confidence across rounds (convergence penalty: -0.05)

### Walkthrough Mode
- Friction scores replace traditional agreement rates
- base_confidence = (average_friction_for_step - 1) / 4 for friction-based findings
- Steps with average friction >= 3.5 automatically get base_confidence >= 0.625

### Comparative Mode
- Agreement rate is calculated per-target
- A finding about Target A doesn't count toward Target B's confidence
- Cross-target findings (true for all targets) get diversity bonus

---

## Synthesis Template Addition

Add this section to the synthesis report:

```markdown
## Confidence-Weighted Findings

### Very High Confidence (>0.85)
- {finding} — {score} [{details}]

### High Confidence (0.70-0.85)
- {finding} — {score} [{details}]

### Medium Confidence (0.50-0.70)
- {finding} — {score} [{details}]

### Low Confidence (0.30-0.50)
- {finding} — {score} [{details}]

### Very Low Confidence (<0.30)
- {finding} — {score} [{details}]

**Confidence Distribution:** {count per tier}
**Median Confidence:** {median score}
```
