# Codebase Review Protocol

This protocol defines 9 phases for thorough codebase review. Follow every phase in order. Do not skip phases. If a tool is unavailable, note the gap and continue.

## Evidence Rules

- NEVER assert without tool output to cite
- "This function is complex" is WORTHLESS — "cyclomatic complexity = 24 (threshold: 10)" is actionable
- Every finding must include the command that produced the evidence
- Prohibited phrases: "could be improved", "seems good", "looks fine", "might have issues"

---

## Phase 0: Orientation (5 minutes)

Get the structural skeleton before reading any code.

**Actions:**
```bash
# Project size and language mix
find . -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.rs" -o -name "*.go" | grep -v node_modules | grep -v dist | wc -l

# Directory structure (depth 3)
find . -maxdepth 3 -type d | grep -v node_modules | grep -v .git | grep -v __pycache__ | sort

# Most-changed files (churn = complexity + risk)
git log --since="6 months ago" --name-only --format="" 2>/dev/null | sort | uniq -c | sort -rn | head -20

# Contributor concentration (bus factor)
git shortlog -sn --since="1 year ago" 2>/dev/null | head -10
```

**What to note:** Total file count, primary language, top churn files (start your deep review here), bus factor risk.

**Minimum evidence:** File count, language, top 5 churn files, contributor distribution.

---

## Phase 1: Build & Type System Health

A broken build invalidates the entire review. Run this first.

**Actions:**
```bash
# TypeScript
npx tsc --noEmit 2>&1 | tail -10
npx tsc --noEmit 2>&1 | grep "error TS" | wc -l

# Python
python -m py_compile src/**/*.py 2>&1 | head -10
mypy --strict src/ 2>&1 | tail -5

# Rust
cargo check 2>&1 | tail -10

# Go
go build ./... 2>&1 | tail -10

# Java/Kotlin
./gradlew compileJava 2>&1 | grep -E "error:|warning:" | wc -l
```

**Thresholds:**
- 0 build errors = PASS (hard gate)
- Any build errors = BLOCKER — note count and stop here until resolved

**Minimum evidence:** Build command output, error count, pass/fail status.

---

## Phase 2: Static Analysis & Complexity

**Linting:**
```bash
# JavaScript/TypeScript
npx eslint --format json . 2>/dev/null | head -100

# Python
ruff check --output-format=json . 2>/dev/null | head -100
flake8 --max-complexity=10 --statistics --count . 2>&1 | tail -10
```

**Complexity measurement:**
```bash
# Python — cyclomatic complexity
radon cc -s -a -n C . 2>/dev/null | head -20

# JavaScript — via eslint complexity rule
npx eslint --rule '{"complexity": ["error", {"max": 10}]}' --no-eslintrc --format compact . 2>/dev/null | wc -l
```

**LSP diagnostics (if available):**
```
lsp_diagnostics_directory(directory_path="src/")
```
Filter by severity=1 (errors) and severity=2 (warnings). Report counts.

**Thresholds:**
- Cyclomatic complexity >10 = refactor candidate, >20 = blocker
- Cognitive complexity >15 = needs attention

**Minimum evidence:** Lint issue count with top 5 offending rules, complexity scores for worst functions.

---

## Phase 3: Security & Dependency Audit

**Dependency vulnerabilities:**
```bash
# Node.js
npm audit 2>/dev/null | tail -15

# Python
pip-audit 2>/dev/null | head -20

# Rust
cargo audit 2>/dev/null | head -20
```

**Outdated dependencies:**
```bash
npm outdated 2>/dev/null | head -20
```

**Unused dependencies:**
```bash
npx depcheck 2>/dev/null | head -20
```

**AST pattern search for security anti-patterns:**
```
ast_grep_search(pattern="eval($X)", language="javascript")
ast_grep_search(pattern="$X as any", language="typescript")
ast_grep_search(pattern="pickle.loads($X)", language="python")
ast_grep_search(pattern="yaml.load($X)", language="python")
```

**Thresholds:**
- Critical CVEs = BLOCKER
- High CVEs = HIGH severity finding
- Unused dependencies = MEDIUM (attack surface reduction)

**Minimum evidence:** CVE count by severity, outdated count, unused deps, AST pattern match counts.

---

## Phase 4: Test Coverage

Line coverage is a vanity metric. Focus on branch coverage and untested error paths.

**Actions:**
```bash
# Jest (Node.js)
npx jest --coverage --passWithNoTests 2>/dev/null | tail -20

# pytest (Python)
pytest --cov=src --cov-report=term-missing -q 2>/dev/null | tail -20

# Rust
cargo tarpaulin --out stdout 2>/dev/null | tail -10

# Go
go test -cover ./... 2>/dev/null | tail -10
```

**Test-to-source ratio:**
```bash
echo "Test files:"; find . -name "*.test.*" -o -name "*_test.*" -o -name "*.spec.*" | grep -v node_modules | wc -l
echo "Source files:"; find src/ -name "*.ts" -o -name "*.py" -o -name "*.rs" | grep -v test | wc -l
```

**Check for untested error paths:**
```bash
grep -rn "catch\|except\|\.catch(" src/ --include="*.ts" --include="*.py" | wc -l
```

**Thresholds:**
- Branch coverage <40% = RED (blocker for critical paths)
- Branch coverage 40-70% = YELLOW
- Branch coverage >70% = GREEN
- Test:source ratio <0.2 = undertesting signal

**Minimum evidence:** Coverage percentages (line + branch), test:source ratio, error handler count vs tested count.

---

## Phase 5: Dead Code Detection (LSP)

The most underused technique. `lsp_find_references` consistently finds 5-15% dead code.

**Protocol:**
```
1. lsp_document_symbols(file_path="src/main-module.ts")     # get all exports
2. For each exported function/class:
   lsp_find_references(file_path, line, character)           # check usage
3. If references == 0 or 1 (only definition): dead code candidate
```

**Type safety check:**
```
lsp_hover(file_path, line, character)
# If return type shows "any" or parameter shows "any" -> flag as type boundary hole
```

**Fallback (if LSP unavailable):**
```bash
# Search for exports with no imports
grep -rn "export " src/ --include="*.ts" | head -30
# Then for each exported name, grep for its usage
```

**Minimum evidence:** Count of dead exports, list of specific dead symbols, type safety holes found.

---

## Phase 6: AST Pattern Detection

`ast_grep_search` finds structural anti-patterns that text-based grep misses.

**Quality patterns:**
```
ast_grep_search(pattern="console.log($$$)", language="javascript")          # debug leakage
ast_grep_search(pattern="catch ($E) {}", language="javascript")             # swallowed errors
ast_grep_search(pattern="except $E: pass", language="python")               # swallowed errors
ast_grep_search(pattern="function $N($A,$B,$C,$D,$E,$$$)", language="js")   # parameter bloat (>4 params)
```

**Technical debt markers:**
```bash
grep -rn "TODO\|FIXME\|HACK\|XXX\|TEMP\|KLUDGE" src/ --include="*.ts" --include="*.py" | wc -l
grep -rn "TODO\|FIXME" src/ | head -20
```

**Thresholds:**
- Empty catch/except blocks = HIGH (silently suppress exceptions)
- console.log in production = MEDIUM
- TODO/FIXME >100 = systemic tech debt signal
- >4 function parameters = refactor candidate

**Minimum evidence:** Count per pattern, specific examples of worst offenders.

---

## Phase 7: Architecture Analysis

**Circular dependency detection:**
```bash
npx madge --circular src/ 2>/dev/null
```

**Module coupling (files with most imports):**
```bash
# Count imports per file
grep -rn "^import\|^from" src/ --include="*.ts" --include="*.py" | cut -d: -f1 | sort | uniq -c | sort -rn | head -15
```

**God files (too large):**
```bash
find src/ -name "*.ts" -o -name "*.py" -o -name "*.rs" | xargs wc -l 2>/dev/null | sort -rn | head -15
```

**Thresholds:**
- Any circular dependencies = HIGH
- Files >500 lines = refactor candidate
- Files >1000 lines = blocker
- Files with >15 local imports = high coupling risk

**Minimum evidence:** Circular deps found, top 5 largest files with line counts, top 5 most-coupled files.

---

## Phase 8: Git History Intelligence

**Churn hotspots (highest risk files):**
```bash
git log --since="6 months ago" --name-only --format="" 2>/dev/null | sort | uniq -c | sort -rn | head -20
```

**Bus factor:**
```bash
git shortlog -sn --since="1 year ago" 2>/dev/null | head -10
# If top contributor >60% of commits: bus factor critically low
```

**PR patterns (if gh CLI available):**
```bash
gh pr list --state merged --limit 20 --json title,additions,deletions,changedFiles 2>/dev/null | head -50
```

**Minimum evidence:** Top 5 churn files, bus factor assessment, recent PR size distribution.

---

## Phase 9: CI/CD & Documentation

**CI configuration:**
```bash
ls -la .github/workflows/ .circleci/ .gitlab-ci.yml Jenkinsfile 2>/dev/null
cat .github/workflows/*.yml 2>/dev/null | grep -E "lint|test|coverage|audit|build" | head -20
```

**Documentation coverage:**
```bash
# Check README exists and is non-empty
wc -l README.md 2>/dev/null
# Check for API docs
find . -name "*.md" -path "*/docs/*" | wc -l
```

**Quality gates in CI:**
- Does CI run linting? tests? coverage checks? security audit?
- Are there coverage thresholds enforced?
- Is there a required review policy?

**Minimum evidence:** CI tools present, quality gates enforced, documentation file count.

---

## Quantitative Review Scorecard

Run this at the end. Red = blocker, Yellow = recommendation, Green = pass.

| Check | Red | Yellow | Green |
|-------|-----|--------|-------|
| Build errors | >0 | — | 0 |
| Critical CVEs | >0 | — | 0 |
| High CVEs | — | >0 | 0 |
| Cyclomatic CC | >20 anywhere | 10-20 exists | all <10 |
| Branch coverage | <40% | 40-70% | >70% |
| Dead exports | >20 | 5-20 | <5 |
| Circular deps | any | — | 0 |
| Bus factor | 1 person >80% | 1 person >60% | distributed |
| TODO/FIXME | >100 | 20-100 | <20 |
| God files (>500L) | >3 files | 1-3 files | 0 |
| Test:source ratio | <0.2 | 0.2-0.5 | >0.5 |
| CI quality gates | none | partial | lint+test+coverage |

---

## Finding Format

Every finding must follow this structure:

**[FINDING]** {what is wrong}
**[EVIDENCE]** {exact command output / tool result that proves it}
**[SEVERITY]** Critical / High / Medium / Low
**[SCOPE]** {N files / N functions / N% of codebase affected}
**[EFFORT]** {estimated fix time: XS <1h / S 1-4h / M 1-3d / L 1-2w}
**[RECOMMENDATION]** {specific action with expected outcome}

---

## Fallback Behavior

| Tool | If Unavailable |
|------|---------------|
| LSP tools | Skip Phases 5 and parts of 2; note gap |
| ast_grep_search | Use grep-based pattern search as fallback |
| npm/pip/cargo tools | Skip if not the project's language; note gap |
| gh CLI | Skip PR analysis; note gap |
| Coverage tools | Note "coverage data unavailable" |
| madge | Skip circular dep check; note gap |
