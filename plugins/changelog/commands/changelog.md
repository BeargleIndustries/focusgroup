---
description: Generate changelogs and audit documentation against code for inconsistencies
argument-hint: [--since TAG|COMMIT] [--audit] [--full] [--fix] [--format md|json] [--out <dir>] [--scope <path>] [--audience dev|user|both] [--group-by type|scope|component]
aliases: [doccheck, doc-check]
---

# Changelog Command

[CHANGELOG ACTIVATED - DOC AUDIT MODE]

You are now running a Changelog + Doc Audit session. This generates changelogs from git history and cross-references all documentation against the actual codebase to catch inconsistencies — wrong install instructions, outdated API examples, nonexistent CLI flags, broken paths, and more.

## User Input

{{ARGUMENTS}}

## Orchestration Protocol

Execute these phases in order. Announce progress at each phase boundary.

---

### Phase 1: Parse Input

Extract from `{{ARGUMENTS}}`:

- **since**: `--since TAG|COMMIT|DATE` (default: last tag, or last 50 commits if no tags)
- **audit_only**: `--audit` (default: false — skip changelog, run only doc audit)
- **full**: `--full` (default: false — changelog + comprehensive doc audit)
- **fix**: `--fix` (default: false — attempt to auto-fix inconsistencies found)
- **format**: `--format md|json` (default: md)
- **out_dir**: `--out <dir>` (default: `.changelog/{YYYY-MM-DD_HH-MM}`)
- **scope**: `--scope <path>` (default: repo root — limit audit to a subdirectory)
- **audience**: `--audience dev|user|both` (default: both)
- **group_by**: `--group-by type|scope|component` (default: type)

If no flags at all, run both changelog generation and doc audit (the default useful mode).

Announce: `[CHANGELOG] Mode: {changelog|audit|full} | Since: {since} | Scope: {scope} | Format: {format}`

---

### Phase 2: Gather Context

**Step 1: Git history**

Unless `--audit` only, collect git log:

```bash
git log --oneline --no-merges {since}..HEAD
git log --format="%H|%s|%an|%ad|%D" --no-merges {since}..HEAD
git diff --stat {since}..HEAD
```

Parse commits into categories using conventional commit prefixes (feat, fix, docs, refactor, perf, test, chore, breaking). Commits without prefixes get classified by content analysis.

**Step 2: Find all documentation files**

```
Glob("**/*.md")
Glob("**/README*")
Glob("**/CONTRIBUTING*")
Glob("**/CHANGELOG*")
Glob("**/*.mdx")
Glob("**/docs/**/*")
Glob("**/*.yaml", "**/*.yml")  # config examples
Glob("**/examples/**/*")
```

Filter to {scope} if set.

**Step 3: Find code entry points**

```
Glob("**/package.json")
Glob("**/setup.py", "**/pyproject.toml", "**/Cargo.toml", "**/go.mod")
Glob("**/Makefile", "**/Dockerfile", "**/docker-compose*")
Glob("**/*.config.*", "**/*.conf")
Glob("**/bin/**/*", "**/cli.*", "**/main.*")
```

Announce: `[CHANGELOG] Found {N} commits, {M} doc files, {K} entry points`

---

### Phase 3: Setup Output Directory

```bash
mkdir -p {out_dir}
```

---

### Phase 4: Generate Changelog (skip if --audit)

Spawn a changelog generator agent:

```
Agent(subagent_type="changelog:changelog-generator", model="sonnet", prompt="
[CHANGELOG_GENERATION]
Generate a changelog from the git history provided.

GIT LOG:
{parsed commit list}

FILES CHANGED:
{git diff stat}

GROUPING: {group_by}
AUDIENCE: {audience}

Rules:
- Group changes by {group_by} with clear headings
- For 'user' audience: focus on visible behavior changes, skip internal refactors
- For 'dev' audience: include all technical changes
- For 'both': include everything, but lead each section with user-facing impact
- Flag BREAKING CHANGES prominently at the top
- Link commit hashes where useful
- Use keep-a-changelog format: Added, Changed, Deprecated, Removed, Fixed, Security
- Be specific: 'Fixed crash when input contains unicode' not 'Fixed bug'
- Include migration notes for breaking changes

Write to: {out_dir}/CHANGELOG.md
")
```

Announce: `[CHANGELOG] Changelog generated with {N} entries`

---

### Phase 5: Doc-Code Consistency Audit

This is the core value. Spawn parallel auditor agents by category.

**Step 5a: Categorize documentation files**

Group doc files by what they claim:

| Category | What to check |
|----------|---------------|
| **Install/Setup** | README install steps, setup guides, dependency lists |
| **API/Usage** | API docs, code examples, function signatures, CLI flags |
| **Config** | Config file examples, environment variables, defaults |
| **Architecture** | System diagrams, component descriptions, file structure claims |
| **Examples** | Code snippets, tutorials, example configs |

**Step 5b: Spawn auditor agents (parallel, by category)**

For each category with files, spawn:

```
Agent(subagent_type="changelog:doc-auditor", model="sonnet", run_in_background=true, prompt="
[DOC_AUDIT:{category}]
You are a Documentation Auditor specializing in {category} documentation.

Your job is to catch documentation that DOES NOT MATCH the actual code. AI coding agents
are notorious for writing docs that look plausible but won't actually work. You are the
defense against this.

DOCUMENTATION FILES TO AUDIT:
{list of files in this category}

APPROACH — Read each doc file, then verify EVERY testable claim against the code:

### Install/Setup Claims
- Do the install commands actually work? Check package names exist in package.json/requirements.txt/etc.
- Do referenced binaries/scripts exist at the paths mentioned?
- Are version requirements accurate?
- Do setup steps execute in the order described?
- Are prerequisites listed that are actually needed? Are needed ones missing?

### API/Usage Claims
- Do function/method signatures match what's in the actual source code?
- Do the documented parameters exist? Are types correct?
- Are documented return values accurate?
- Do code examples use APIs that actually exist and have correct syntax?
- Are CLI flags/commands real? Check against argument parsers, commander configs, etc.
- Are documented defaults actually the defaults in code?

### Config Claims
- Do documented config keys exist in the code that reads config?
- Are documented default values the actual defaults?
- Do environment variable names match what the code reads?
- Are example config files valid for the schema the code expects?

### Architecture Claims
- Do documented file paths exist?
- Do described components/modules exist?
- Are dependency relationships accurate?
- Do described data flows match the actual code flow?

### Example Claims
- Do code snippets compile/parse correctly?
- Do examples reference real exports, functions, classes?
- Are import paths correct?
- Would the example actually produce the described output?

TOOLS: Use Read, Grep, Glob, and Bash to verify everything. Check actual source files.

For EACH inconsistency found, record:
- **File**: doc file path and line number
- **Claim**: what the doc says (quote it)
- **Reality**: what the code actually does (cite source file and line)
- **Severity**: CRITICAL (will break user's setup), HIGH (misleading), MEDIUM (outdated), LOW (cosmetic)
- **Suggestion**: what the doc should say instead

Also note documentation GAPS — things the code does that aren't documented at all,
especially new features from recent commits.

Write findings to: {out_dir}/audit-{category_slug}.md

Format:
# Doc Audit: {Category}
**Files audited:** {count}
**Inconsistencies found:** {count}
**Severity breakdown:** {critical}C / {high}H / {medium}M / {low}L

## Inconsistencies

### 1. {Brief title}
- **File:** {path}:{line}
- **Claim:** \"{quoted text}\"
- **Reality:** {what code actually does} (see `{source_file}:{line}`)
- **Severity:** {CRITICAL|HIGH|MEDIUM|LOW}
- **Fix:** {suggested correction}

## Documentation Gaps
{list of undocumented features/behaviors}

## Verified Claims
{count} claims checked and confirmed accurate
")
```

Wait for all auditors to complete.

Announce: `[CHANGELOG] Doc audit complete — {N} inconsistencies found across {M} categories`

---

### Phase 6: Cross-Reference Changelog with Docs

If both changelog and audit ran, spawn a cross-reference check:

```
Agent(subagent_type="changelog:inconsistency-reporter", model="sonnet", prompt="
[CROSS_REFERENCE]
Check whether recent code changes have corresponding documentation updates.

CHANGELOG: Read {out_dir}/CHANGELOG.md
AUDIT RESULTS: Read all {out_dir}/audit-*.md files
DOC FILES: {list of all doc files}

For each significant change in the changelog:
1. Does any documentation need updating because of this change?
2. Was the documentation actually updated? (check git diff for doc file changes)
3. If not, flag it as a documentation gap.

Also look for:
- New exports/functions/commands added without docs
- Changed function signatures where docs still show old signature
- Removed features still documented
- New config options without documentation
- Changed defaults without doc updates

Write to: {out_dir}/cross-reference.md

Format:
# Cross-Reference: Changes vs Documentation

## Changes Missing Documentation
| Change | Type | Files Affected | Doc Status |
|--------|------|---------------|------------|
| {description} | {feat/fix/etc} | {files} | {missing/outdated/ok} |

## Stale Documentation
{docs that reference things changed or removed in this period}

## Summary
- Changes with adequate docs: {N}
- Changes missing docs: {N}
- Docs made stale by changes: {N}
")
```

---

### Phase 7: Synthesize Results

Spawn a synthesis agent:

```
Agent(subagent_type="changelog:inconsistency-reporter", model="sonnet", prompt="
[SYNTHESIS]
Read ALL files in {out_dir}/ and produce a unified report.

Write to: {out_dir}/report.md

Format:
# Changelog & Doc Audit Report
**Date:** {date} | **Scope:** {scope} | **Since:** {since}

## Executive Summary
{2-3 sentences: how many changes, how many doc issues, overall health}

## Changelog Highlights
{Top 5 most significant changes, with breaking changes first}

## Documentation Health Score
**Score: {0-100}/100**

Scoring:
- Start at 100
- -10 per CRITICAL inconsistency
- -5 per HIGH inconsistency
- -2 per MEDIUM inconsistency
- -1 per LOW inconsistency
- -3 per undocumented significant change
- Minimum 0

**{score >= 80: 'HEALTHY' | score >= 50: 'NEEDS ATTENTION' | score < 50: 'CRITICAL'}**

## Critical Issues (Fix Immediately)
{CRITICAL and HIGH severity items, grouped by file}

## Maintenance Items
{MEDIUM and LOW items}

## Documentation Gaps
{Features/changes without corresponding docs}

## Recommendations
{Prioritized list of what to fix first}
")
```

---

### Phase 7.5: Auto-Fix (if --fix)

**Only if `--fix` flag is set.** For each CRITICAL and HIGH inconsistency with a clear fix:

```
Agent(subagent_type="changelog:doc-fixer", model="sonnet", prompt="
[DOC_FIX]
Read the audit report at {out_dir}/report.md.

For each CRITICAL and HIGH severity inconsistency that has a clear, unambiguous fix:
1. Read the documentation file
2. Apply the fix using the Edit tool
3. Record what you changed

RULES:
- Only fix documentation, NEVER modify source code
- Only fix when the correct value is unambiguous (e.g., wrong function name, wrong path)
- Skip fixes that require judgment calls (e.g., rewriting explanations)
- If a doc section is fundamentally wrong, flag it for human review instead of guessing

Write a fix log to: {out_dir}/fixes-applied.md

Format:
# Fixes Applied

## Auto-Fixed ({count})
| File | Line | What Changed | Severity |
|------|------|-------------|----------|
| {path} | {line} | {old} -> {new} | {severity} |

## Skipped (Needs Human Review) ({count})
| File | Line | Issue | Why Skipped |
|------|------|-------|-------------|
| {path} | {line} | {issue} | {reason} |
")
```

Announce: `[CHANGELOG] Auto-fixed {N} issues, {M} flagged for human review`

---

### Phase 8: JSON Output (if --format json)

If `--format json`, write `{out_dir}/report.json`:

```json
{
  "meta": {
    "date": "{ISO date}",
    "since": "{since}",
    "scope": "{scope}",
    "commits_analyzed": N,
    "docs_audited": N
  },
  "changelog": {
    "breaking": [...],
    "added": [...],
    "changed": [...],
    "fixed": [...],
    "removed": [...]
  },
  "audit": {
    "health_score": N,
    "health_status": "HEALTHY|NEEDS ATTENTION|CRITICAL",
    "inconsistencies": [
      {
        "file": "{path}",
        "line": N,
        "claim": "{quoted}",
        "reality": "{actual}",
        "severity": "CRITICAL|HIGH|MEDIUM|LOW",
        "fix": "{suggestion}"
      }
    ],
    "gaps": [...],
    "verified_claims": N
  },
  "cross_reference": {
    "changes_missing_docs": N,
    "stale_docs": N,
    "changes_with_docs": N
  },
  "fixes_applied": []
}
```

---

### Phase 9: Report

```
[CHANGELOG COMPLETE]

Since: {since}
Commits: {N}
Docs Audited: {M} files

Documentation Health: {score}/100 ({status})

Inconsistencies Found:
  CRITICAL: {N}
  HIGH: {N}
  MEDIUM: {N}
  LOW: {N}

{If changelog generated:}
Changelog: {N} entries ({breaking} breaking changes)

{If cross-reference ran:}
Changes missing docs: {N}
Stale documentation: {N}

{If fixes applied:}
Auto-fixed: {N}
Needs human review: {M}

Top Issues:
{1-3 most critical findings with file:line references}

Output: {out_dir}/
  - report.md (full report)
  - CHANGELOG.md (changelog)
  - audit-*.md (category audit details)
  - cross-reference.md (change-to-doc mapping)
  {- fixes-applied.md (if --fix)}
  {- report.json (if --format json)}

Read {out_dir}/report.md for the full analysis.
```

---

## Error Recovery

| Error | Action |
|-------|--------|
| No git history available | Error: "Not a git repository or no commits found" |
| No tags found for --since default | Fall back to last 50 commits |
| No documentation files found | Warn, generate changelog only |
| Individual auditor agent fails | Skip category, note gap in report |
| --fix fails on a file | Log failure, continue with others |
| Scope path doesn't exist | Error: "Scope path {path} not found" |
| Git log parsing fails | Fall back to simple `git log --oneline` |

## Cancellation

User can cancel at any time. Partial results remain in the output directory.
