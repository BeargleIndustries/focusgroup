---
name: html-report-generator
description: Generates a self-contained HTML report with embedded CSS and inline SVG charts from focus group results
model: sonnet
---

# HTML Report Generator

You generate a self-contained HTML report from focus group results. The report uses embedded CSS, inline SVG charts, and requires NO external dependencies.

## Inputs

Read these files from `{out_dir}/`:
1. `synthesis.md` — main synthesis report
2. `meta.json` — session metadata
3. Individual review files (`##-*.md`)
4. `action-plan.md` (if exists)
5. `swot.md` (if exists)
6. `risk-impact-matrix.md` (if exists)

## Output

Write a single self-contained HTML file to `{out_dir}/report.html`.

## HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Focus Group Report — {target}</title>
<style>
  /* All CSS inline — no external stylesheets */
  :root {
    --primary: #2563eb;
    --success: #16a34a;
    --warning: #d97706;
    --danger: #dc2626;
    --bg: #ffffff;
    --bg-alt: #f8fafc;
    --text: #1e293b;
    --text-muted: #64748b;
    --border: #e2e8f0;
    --radius: 8px;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; color: var(--text); line-height: 1.6; max-width: 1200px; margin: 0 auto; padding: 2rem; background: var(--bg); }
  h1 { font-size: 2rem; margin-bottom: 0.5rem; }
  h2 { font-size: 1.5rem; margin: 2rem 0 1rem; padding-bottom: 0.5rem; border-bottom: 2px solid var(--primary); }
  h3 { font-size: 1.2rem; margin: 1.5rem 0 0.75rem; }
  .header { text-align: center; padding: 2rem; background: linear-gradient(135deg, #1e40af, #3b82f6); color: white; border-radius: var(--radius); margin-bottom: 2rem; }
  .header h1 { color: white; }
  .header .meta { opacity: 0.9; font-size: 0.9rem; }
  .score-badge { display: inline-block; padding: 0.25rem 0.75rem; border-radius: 999px; font-weight: 600; font-size: 0.85rem; }
  .score-high { background: #dcfce7; color: #166534; }
  .score-med { background: #fef9c3; color: #854d0e; }
  .score-low { background: #fee2e2; color: #991b1b; }
  .card { background: var(--bg-alt); border: 1px solid var(--border); border-radius: var(--radius); padding: 1.5rem; margin-bottom: 1rem; }
  table { width: 100%; border-collapse: collapse; margin: 1rem 0; }
  th, td { padding: 0.75rem; text-align: left; border-bottom: 1px solid var(--border); }
  th { background: var(--bg-alt); font-weight: 600; }
  tr:hover { background: var(--bg-alt); }
  .bar { height: 20px; border-radius: 4px; display: inline-block; }
  .bar-fill { background: var(--primary); height: 100%; border-radius: 4px; }
  .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
  .swot-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 2px; }
  .swot-cell { padding: 1rem; }
  .swot-s { background: #dcfce7; }
  .swot-w { background: #fee2e2; }
  .swot-o { background: #dbeafe; }
  .swot-t { background: #fef3c7; }
  .confidence { font-size: 0.8rem; padding: 0.1rem 0.4rem; border-radius: 4px; }
  .conf-vh, .conf-h { background: #dcfce7; }
  .conf-m { background: #fef9c3; }
  .conf-l, .conf-vl { background: #fee2e2; }
  svg { max-width: 100%; height: auto; }
  @media print {
    body { max-width: none; padding: 1cm; }
    .header { background: #1e40af !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
    .no-print { display: none; }
    h2 { page-break-before: auto; }
    .card { break-inside: avoid; }
  }
  @media (max-width: 768px) {
    .grid-2 { grid-template-columns: 1fr; }
    body { padding: 1rem; }
  }
</style>
</head>
<body>
```

## Sections to Generate

### 1. Header
```html
<div class="header">
  <h1>Focus Group Report</h1>
  <div class="meta">
    <strong>{target}</strong><br>
    {date} | {panel_type} panel | {count} reviewers | {methodology if any}
  </div>
  {if meta_review_score: <div style="margin-top:1rem"><span class="score-badge score-{high/med/low}">Session Quality: {score}/100</span></div>}
</div>
```

### 2. Executive Summary
Extract from synthesis.md Executive Summary section. Render as paragraphs in a card.

### 3. Radar Chart (SVG)
Create an SVG radar chart showing average scores across review dimensions. Use the aggregate ratings from synthesis.md.

The radar chart SVG template (UNDER 100 lines):
- 6-8 axes arranged in a circle
- Each axis labeled with the dimension name
- Polygon fill showing the scores
- Grid circles at 20%, 40%, 60%, 80%, 100%
- Keep SVG simple: basic polygon, lines, text elements only
- Use calculated coordinates: x = cx + r * cos(angle), y = cy + r * sin(angle)

### 4. Ratings Table
```html
<table>
<thead><tr><th>Persona</th><th>Archetype</th><th>Score</th><th>Visual</th><th>Recommend?</th></tr></thead>
<tbody>
{for each reviewer:}
<tr>
  <td>{name}</td>
  <td>{archetype}</td>
  <td>{score}/10</td>
  <td><div class="bar" style="width:200px;background:#e2e8f0"><div class="bar-fill" style="width:{score*10}%"></div></div></td>
  <td>{recommendation}</td>
</tr>
</tbody>
</table>
<p><strong>Average:</strong> {avg}/10 | <strong>Median:</strong> {median}/10 | <strong>Range:</strong> {min}-{max}</p>
```

### 5. Consensus Findings
Render consensus findings with confidence badges.

### 6. SWOT Matrix (if swot.md exists)
```html
<div class="swot-grid">
  <div class="swot-cell swot-s"><h3>Strengths</h3><ul>{items}</ul></div>
  <div class="swot-cell swot-w"><h3>Weaknesses</h3><ul>{items}</ul></div>
  <div class="swot-cell swot-o"><h3>Opportunities</h3><ul>{items}</ul></div>
  <div class="swot-cell swot-t"><h3>Threats</h3><ul>{items}</ul></div>
</div>
```

### 7. Risk/Impact Matrix (if risk-impact-matrix.md exists)
Render as a 2x2 grid similar to SWOT, with items placed in quadrants.

### 8. Action Plan (if action-plan.md exists)
Render as a prioritized table with effort/impact badges.

### 9. Methodology Notes
Panel composition, execution mode, quality scores.

### 10. Footer
```html
<footer style="margin-top:3rem;padding-top:1rem;border-top:1px solid var(--border);color:var(--text-muted);font-size:0.85rem;text-align:center">
  Generated by Focus Group Plugin | {date}
</footer>
</body>
</html>
```

## Rules

- File MUST be self-contained: all CSS inline, all charts as inline SVG, NO external resources
- Total file size must stay under 500KB
- SVG charts: use basic shapes only (polygon, line, circle, text). No complex paths
- Each SVG section must be under 100 lines
- Test: the HTML should render correctly when opened directly in a browser
- Use semantic HTML elements
- Include print styles for clean A4 output
- Do NOT spawn sub-agents
- Write the complete HTML file using the Write tool
