# Website Review Protocol

This protocol defines 8 phases for thorough website review. Follow every phase in order. Do not skip phases. Collect evidence at each phase before moving to the next.

## Evidence Rules

- Every finding MUST include specific evidence (URL, metric value, screenshot reference, or direct quote)
- Use these prohibited phrases NEVER — replace with specifics: "could be improved", "seems good", "looks fine", "might have issues", "generally well"
- Quantify everything: load times in ms, counts of elements, specific dimensions

---

## Phase 1: Discovery

Before reviewing any page in detail, map the site structure.

**Actions:**
1. Check `{target}/robots.txt` — use WebFetch or navigate. Note: disallowed paths, sitemap references
2. Check `{target}/sitemap.xml` — note page count, last modified dates
3. On the homepage, harvest all navigation links using `mcp__claude-in-chrome__javascript_tool`:
   ```javascript
   JSON.stringify(Array.from(document.querySelectorAll('nav a, footer a, header a')).map(a => ({text: a.textContent.trim(), href: a.href})))
   ```
4. Categorize links: primary nav, footer, CTA buttons, internal, external

**Minimum evidence:** Complete link inventory with categorization.

---

## Phase 2: Navigation Structure

Visit at least 5 key pages (homepage + 4 from nav). On EACH page collect:

**Actions per page:**
1. Run the website probe (if probe data was provided, reference it; otherwise run the JS probe inline)
2. Verify: page title exists and is unique, H1 exists and matches page purpose
3. Check: canonical URL tag present, breadcrumbs where expected
4. Check: no dead links (note any 404s encountered)
5. Test: back button behavior works correctly

**Minimum evidence:** Title, H1, canonical status for each visited page.

---

## Phase 3: Interactive Elements

Test every interactive element you find.

**Forms:**
1. Find all forms: `document.querySelectorAll('form')`
2. For EACH form:
   - Submit empty (test validation messages — should NOT get a 500 error)
   - Enter invalid data (wrong email format, etc.) — verify inline validation
   - Enter valid test data — verify success state
   - Check: error messages linked to fields (aria-describedby or proximity)
   - Check: required fields marked (aria-required or required attribute)

**Search:**
1. Find search input: `input[type="search"], input[placeholder*="search" i]`
2. Test with a real query — verify results appear
3. Test with nonsense query — verify "no results" state exists

**Modals/Overlays:**
1. Find modal triggers and click them
2. Verify: focus moves into modal, Escape closes it, focus returns to trigger

**Dropdowns/Menus:**
1. Test hover and click states on navigation menus
2. Verify all submenu items are reachable via keyboard (Tab/arrow keys)

**Minimum evidence:** Form test results, search behavior, modal/menu behavior documented.

---

## Phase 4: Performance

Collect quantitative performance data.

**Actions:**
1. If probe data is available, reference the performance section
2. Otherwise, run via `mcp__claude-in-chrome__javascript_tool`:
   ```javascript
   (() => {
     const nav = performance.getEntriesByType('navigation')[0];
     const resources = performance.getEntriesByType('resource');
     return JSON.stringify({
       ttfb: nav ? Math.round(nav.responseStart - nav.requestStart) + 'ms' : 'unavailable',
       domComplete: nav ? Math.round(nav.domComplete) + 'ms' : 'unavailable',
       loadEvent: nav ? Math.round(nav.loadEventEnd) + 'ms' : 'unavailable',
       resourceCount: resources.length,
       totalTransferKB: Math.round(resources.reduce((s,r) => s + (r.transferSize||0), 0) / 1024),
       largestResources: resources.sort((a,b) => (b.transferSize||0) - (a.transferSize||0)).slice(0,5).map(r => ({
         name: r.name.split('/').pop()?.slice(0,50),
         sizeKB: Math.round((r.transferSize||0)/1024),
         type: r.initiatorType
       }))
     }, null, 2);
   })()
   ```
3. Check network requests: `mcp__claude-in-chrome__read_network_requests` — note failed requests, third-party domains
4. Check console: `mcp__claude-in-chrome__read_console_messages` — note errors, warnings, CSP violations

**Thresholds (Core Web Vitals 2025):**
- LCP: Good <2.5s, Needs Work 2.5-4s, Poor >4s
- Total page weight: Excellent <1MB, Acceptable 1-3MB, Poor >3MB
- HTTP requests: Excellent <50, Acceptable 50-100, Poor >100

**Minimum evidence:** TTFB, DOM complete time, total page weight, resource count, any console errors.

---

## Phase 5: Accessibility

Check WCAG 2.1 Level AA compliance.

**Automated checks** (via probe data or inline JS):
1. Images without alt text — count
2. Form inputs without labels — count
3. Heading hierarchy — verify sequential (no H1→H3 jumps)
4. Skip link present — yes/no
5. Language attribute on html — present and correct
6. ARIA landmarks — main, nav, banner, contentinfo present
7. Buttons/links without accessible text — count
8. Focus indicators — Tab through the page, verify visible focus on each element

**Keyboard navigation test:**
1. Use `mcp__claude-in-chrome__shortcuts_execute` with Tab key 10+ times
2. Verify: focus is visible at each step
3. Verify: logical tab order (top-left to bottom-right flow)
4. Test Shift+Tab for reverse
5. If modals/menus exist: verify focus trapping inside them

**Minimum evidence:** Count of images without alt, inputs without labels, heading hierarchy diagram, skip link status, keyboard navigation pass/fail.

---

## Phase 6: SEO

Evaluate search engine optimization signals.

**Technical SEO** (from probe data or inline JS):
1. Title tag: exists, 50-60 characters ideal
2. Meta description: exists, 150-160 characters ideal
3. Canonical URL: present and correct
4. Open Graph tags: title, description, image present
5. Structured data (JSON-LD): types present
6. H1: exactly 1 per page
7. Internal links: count (more = better for SEO)
8. Image alt text coverage: ratio of images with meaningful alt text

**Additional checks:**
- robots.txt: Sitemap directive present
- Pages that should be indexed aren't noindexed
- No orphan pages (pages not linked from navigation)

**Minimum evidence:** Title/description length, OG tag presence, structured data types, H1 count, image alt ratio.

---

## Phase 7: Mobile Responsiveness

Test at 3 viewport sizes.

**Actions:**
1. `mcp__claude-in-chrome__resize_window` to 375x812 (mobile)
   - Take screenshot: `mcp__claude-in-chrome__computer`
   - Check horizontal scroll: `document.body.scrollWidth > window.innerWidth`
   - Verify: hamburger/mobile nav works, text is readable
   - Check touch targets: buttons/links at least 44x44px

2. `mcp__claude-in-chrome__resize_window` to 768x1024 (tablet)
   - Take screenshot
   - Verify: layout adapts (not just scaled down desktop)

3. `mcp__claude-in-chrome__resize_window` to 1440x900 (desktop)
   - Take screenshot
   - Verify: content uses space appropriately, not stretched to full width

**Touch target check:**
```javascript
JSON.stringify({
  smallTargets: Array.from(document.querySelectorAll('a, button')).filter(el => {
    const r = el.getBoundingClientRect();
    return r.width > 0 && r.height > 0 && (r.width < 44 || r.height < 44);
  }).length
})
```

**Minimum evidence:** Screenshots at 3 viewports, horizontal scroll status, touch target count below 44px.

---

## Phase 8: Security Surface

Check client-side security posture.

**Actions:**
1. HTTPS: verify `location.protocol === 'https:'`
2. Mixed content: check for `http:` resources loaded on HTTPS page
3. External scripts: list third-party script domains (supply chain risk)
4. Subresource Integrity (SRI): count of scripts with `integrity` attribute
5. Security headers: check via `mcp__claude-in-chrome__read_network_requests` for:
   - Strict-Transport-Security (HSTS)
   - Content-Security-Policy (CSP)
   - X-Content-Type-Options
   - X-Frame-Options
   - Referrer-Policy
6. Cookie exposure: any cookies accessible via `document.cookie` (missing HttpOnly flag)
7. Console CSP violations: check `mcp__claude-in-chrome__read_console_messages` filtered for security

**Minimum evidence:** HTTPS status, mixed content count, external script domains, security headers present/missing.

---

## Minimum Viable Review Checklist

If time-constrained, these 12 checks deliver ~80% of value:

1. [ ] LCP measurement (homepage) — quantified in ms
2. [ ] Mobile viewport test at 375px — screenshot
3. [ ] All nav links functional — broken link count
4. [ ] Forms tested empty + invalid — validation behavior
5. [ ] Images with alt text — ratio (X of Y)
6. [ ] Skip link present — yes/no
7. [ ] Console errors — count on homepage
8. [ ] HTTPS + HSTS header — yes/no
9. [ ] Meta title/description length — character count
10. [ ] H1 present and unique — per page
11. [ ] Contact/main form end-to-end — success state reached
12. [ ] Page weight — KB total, breakdown by type

---

## Anti-Scraping Guidance

| Situation | Action |
|-----------|--------|
| Cloudflare JS challenge | Wait 10s in real browser; it self-resolves |
| Turnstile/reCAPTCHA on page | Document as finding (accessibility/UX issue); don't attempt to solve |
| CAPTCHA blocking a form | Screenshot as evidence; note form cannot be fully tested |
| Rate limiting (429 errors) | Pace at 2-3s between navigations; sample strategically |
| Content behind login | Note as limitation; review only publicly accessible content |

---

## Finding Format

Every finding must follow this structure:

**[FINDING]** {what is wrong}
**[EVIDENCE]** {specific data: metric value, URL, screenshot ref, element selector}
**[SEVERITY]** Critical / High / Medium / Low / Informational
**[CATEGORY]** Performance / Accessibility / SEO / Security / UX / Content
**[STANDARD]** {reference: WCAG 2.1 SC X.X.X / Core Web Vitals / OWASP}
**[RECOMMENDATION]** {specific action + expected outcome}
