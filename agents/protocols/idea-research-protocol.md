# Idea & Product Research Protocol

This protocol defines 6 phases for evidence-based product/idea review. Follow every phase in order. Do not skip phases. Every major claim must trace to a retrievable source.

## Evidence Rules

- NEVER make a market claim without citing a source
- NEVER say "strong demand" without search volume, review volume, or funding signals
- NEVER say "no real competition" without systematic search across G2, Crunchbase, Product Hunt
- Every claim must be tagged by evidence tier (T1-T5)
- Prohibited phrases: "could be improved", "has potential", "interesting idea", "might work"

## Evidence Tier System

| Tier | Description | Acceptable Use |
|------|-------------|----------------|
| T1 | Primary data (Census, SEC filings, peer-reviewed) | High-confidence market assertions |
| T2 | Industry reports (Gartner, Forrester, Statista, CB Insights) | Market size claims |
| T3 | Aggregated user data (G2, Capterra, App Store ratings) | Customer sentiment claims |
| T4 | Forum/community (Reddit, HN, Quora) | Pain point illustrations |
| T5 | Single source / anecdote | Illustrative only — never sole basis for a claim |

**Rule:** No market size claim without T1 or T2. No pain point claim without T4+ with volume evidence.

---

## Phase 1: Market Sizing (2-3 searches)

Estimate TAM/SAM/SOM using at least 2 independent methods.

**Search query templates:**
```
"{industry} market size" billion 2024 site:statista.com OR site:grandviewresearch.com
"{product category} global market" CAGR forecast 2030
"{vertical} software market size" Gartner OR Forrester OR IDC 2024
```

**Top-Down method:**
1. Find total industry revenue from a T1/T2 source
2. Apply segmentation multipliers to narrow to addressable market
3. Record: TAM = $X.XB ([Source], [Year])

**Bottom-Up method:**
1. Count potential buyers (search: `"number of {customer type}" site:census.gov OR site:bls.gov`)
2. Estimate annual spend from competitor pricing pages
3. Calculate: count × ACV = SAM
4. Apply 0.5-5% capture rate = SOM

**Value-Theory method:**
1. Estimate hours saved per user per year
2. Multiply by hourly rate × number of workers
3. Search: `"cost of {problem}" study OR report`

**Cross-validate:** If methods disagree by >2x, flag as "high uncertainty — range estimate."

**Minimum evidence:** TAM figure with source, growth rate (CAGR), methodology stated.

---

## Phase 2: Competitive Landscape (3-4 searches)

Identify ALL competitors, not just the obvious ones.

**Five search passes (all required):**
```
# Direct competitors
"{solution type}" alternatives 2024 site:g2.com OR site:capterra.com
"best {product category}" comparison site:softwareadvice.com OR site:getapp.com

# Funded competitors (VC-validated)
site:crunchbase.com/organization "{keyword}" seed OR series-a 2023 2024 2025

# Adjacent competitors (may expand into your space)
"{related workflow}" software OR platform OR tool

# Gap mining from reviews
site:g2.com "{top competitor}" review "wish" OR "missing" OR "if only"
site:reddit.com "{top competitor}" "stopped using" OR "switched" OR "looking for alternative"
```

**For top 5-7 competitors, fill this matrix:**

| Dimension | Source |
|-----------|--------|
| Pricing model + entry price | Competitor pricing page |
| Target segment (SMB/mid/enterprise) | Homepage, case studies |
| G2 rating + review count | g2.com/{product} |
| Funding raised + last round date | Crunchbase |
| Employee count (scale proxy) | LinkedIn |
| Core differentiator claim | Homepage H1/hero text |
| Integration ecosystem depth | Integrations page |
| Top user complaint themes | G2 1-3 star reviews |

**Use WebFetch to pull real data from competitor pricing pages and G2 profiles.**

**Minimum evidence:** 5+ competitors identified, 3+ with pricing data, competitive matrix filled.

---

## Phase 3: Customer Pain Points (2-3 searches)

Find REAL user pain points with verbatim quotes, not assumed ones.

**Search channels (in priority order):**

**Tier 1 — Highest signal:**
```
site:reddit.com "{problem domain}" frustrated OR "looking for" OR "anyone use"
site:news.ycombinator.com "Ask HN" "{problem domain}"
```
Key subreddits: r/entrepreneur, r/smallbusiness, r/SaaS, r/startups, plus vertical-specific subs.

**Tier 2 — Structured:**
```
site:quora.com "best way to" OR "how do you" "{workflow}"
"{top competitor}" review site:g2.com 1 star OR 2 star OR 3 star
```
G2/Capterra 3-star reviews are the most honest — user got value but has real complaints.

**Pain classification (tag each pain point found):**
- FUNCTIONAL: "It doesn't do X I need"
- EMOTIONAL: "It feels overwhelming/confusing"
- SOCIAL: "It makes me look bad to clients"
- FINANCIAL: "ROI isn't there / too expensive"
- PROCESS: "Doesn't fit how my team works"
- RELIABILITY: "It breaks / loses data / slow"

**JTBD statement format:**
> When [situation], I want to [motivation], so I can [outcome].

Derive this from the pain point evidence — don't invent it.

**Minimum evidence:** 3+ verbatim user quotes with sources, pain classification, JTBD statement.

---

## Phase 4: Feasibility & Risk Assessment (1-2 searches)

**Technical feasibility:**
- Core technical risks: What's the hardest part to build?
- Build vs buy decisions: Key APIs, infrastructure, data sources needed
- Scalability concerns: What breaks at 10x, 100x usage?

**Regulatory scan:**
```
"{industry}" regulation compliance requirements 2024 2025
"{data type} handling" GDPR OR CCPA OR HIPAA requirements
"{product category}" FDA clearance OR SEC registration OR FTC rules
```

**Red flag categories:**
- Health/medical: FDA 510(k), HIPAA
- Finance/payments: PCI DSS, SOC2, FinCEN
- Children's: COPPA
- AI/data: GDPR, CCPA, EU AI Act
- Real estate: state broker licensing

**IP landscape:**
- Patent search: patents.google.com for core technology concepts
- Trademark: USPTO TESS for proposed product names

**Minimum evidence:** 1+ specific regulatory risk with statute, 1+ technical risk, IP conflict check.

---

## Phase 5: Validation Framework Scoring

Apply structured frameworks to evaluate the idea.

**Value Proposition Canvas:**
1. Map customer jobs, pains, gains (from Phase 3 evidence)
2. Map product's pain relievers and gain creators
3. Score fit: pain reliever → pain matches (X/6)
4. Below 4/6 = positioning problem

**Porter's Five Forces (evidence-based):**

| Force | Evidence Source |
|-------|---------------|
| New entrants | Crunchbase funding activity last 24 months |
| Supplier power | Key API/data dependencies, pricing history |
| Buyer power | Switching cost signals from reviews |
| Substitutes | `"{job}" excel template OR google sheets OR manual process"` |
| Rivalry | Competitor count, price war signals, market growth |

**Market Opportunity Scorecard (0-100):**

```
Size (0-25):    TAM >$10B=25, $1B-10B=20, $100M-1B=12, <$100M=5
Growth (0-20):  CAGR >20%=20, 10-20%=15, 5-10%=8, <5%=3
Fragmentation (0-20): No dominant player=20, 2-3 leaders=12, 1-2 dominant=4
Pain Intensity (0-20): Active WTP=20, tolerated workarounds=12, nice-to-have=4
Defensibility (0-15): Network/data/regulatory moat=15, technical diff=10, execution-only=4

70+: Strong opportunity
50-69: Viable with clear strategy
30-49: Proceed with caution
<30: Significant structural challenges
```

**Minimum evidence:** Value prop fit score, Porter's assessment, opportunity scorecard number.

---

## Phase 6: Analogous Case Studies (2-3 searches)

Find 2 successes and 1 failure in adjacent markets.

**Success case search:**
```
"{adjacent category}" startup success growth story site:firstround.com OR site:a16z.com
"how {company} grew to" million users OR revenue
"{product category}" Y Combinator success 2022 2023 2024
```

**Failure case search:**
```
"{adjacent category}" startup failed OR shutdown postmortem lessons
"why {company} failed" OR "shutting down"
"{product category}" graveyard site:techcrunch.com
```

**For each case study, extract:**
1. Initial hypothesis vs what actually drove growth (or killed it)
2. Key metric at inflection point (CAC, payback period, retention)
3. Specific parallel to the idea being reviewed
4. What the reviewer's idea can learn from this case

**Minimum evidence:** 1 success parallel, 1 failure warning, with specific lessons drawn.

---

## Output Structure

Structure your review findings as:

```markdown
## Market Opportunity
- TAM: $X.XB ([source], [year]) [T2]
- SAM: $X.XM (methodology: bottom-up from [count] × [ACV])
- CAGR: X.X% through [year] ([source]) [T2]
- Opportunity Score: [X]/100

## Competitive Landscape
[Competitor comparison table]
Key gaps: [specific gaps from review data]

## Customer Pain Points
JTBD: "When [situation], I want to [goal], so I can [outcome]"
1. "[Verbatim quote]" — [source, date] [T4]
2. "[Verbatim quote]" — [source, date] [T4]
3. "[Verbatim quote]" — [source, date] [T4]

## Risk Assessment
| Risk | Severity | Evidence | Mitigation |
...

## Case Study Parallels
Success: [Company] — [lesson]
Failure: [Company] — [warning]
```

---

## Finding Format

Every finding must follow this structure:

**[FINDING]** {what was discovered}
**[EVIDENCE]** {source URL, data point, or verbatim quote}
**[EVIDENCE TIER]** T1-T5
**[CONFIDENCE]** High / Medium / Low
**[IMPLICATION]** {what this means for the idea}
**[RECOMMENDATION]** {specific action or decision this informs}
