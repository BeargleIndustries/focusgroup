---
description: Set up Obsidian vault memory — creates vault, scaffolds project, generates CLAUDE.md config
argument-hint: "[--path <vault-path>]"
aliases: [obs-setup]
---

# Obs-Setup Command

[OBS-SETUP ACTIVATED — VAULT INITIALIZATION MODE]

You are now running first-time vault setup. This creates a persistent Obsidian knowledge vault, scaffolds the current project into it, and generates the CLAUDE.md snippet needed to activate auto-orient behavior.

## User Input

{{ARGUMENTS}}

## Execution Protocol

Execute these phases in order. Announce progress at each phase boundary.

---

### Phase 1: Parse Input

Extract from `{{ARGUMENTS}}`:

- **vault_path**: `--path <vault-path>` (default: `~/Documents/AgentMemory`)

Resolve `~` to the user's home directory using the shell: `echo ~` or platform equivalent.

Announce: `[OBS-SETUP] Vault path: {vault_path}`

---

### Phase 2: Check Existing Vault (IDEMPOTENCY)

Check if `{vault_path}/Home.md` exists.

- If it **exists**: announce `[OBS-SETUP] Vault already exists at {vault_path} — skipping creation` and skip to Phase 4.
- If it **does not exist**: proceed to Phase 3.

---

### Phase 3: Create Vault

Create these directories (use `mkdir -p` or equivalent):

- `{vault_path}/projects/`
- `{vault_path}/domains/`
- `{vault_path}/patterns/`
- `{vault_path}/sessions/`
- `{vault_path}/templates/`
- `{vault_path}/todos/`
- `{vault_path}/inbox/`
- `{vault_path}/attachments/`
- `{vault_path}/.obsidian/`

Write these files **only if they don't already exist**. Write template files with placeholder tokens (`{{date}}`, `{Project Name}`, `{short-name}`, etc.) as **literal text** — these are Obsidian templates for future use, not values to interpolate now:

**`{vault_path}/Home.md`:**
```markdown
---
aliases: [Dashboard, Home]
tags: [meta]
type: index
---

# Agent Memory Vault

Welcome to your persistent knowledge vault. This graph-structured memory persists across coding sessions.

## Navigation

- [[projects/Projects|Projects]] — All tracked projects
- [[domains/Domains|Domains]] — Technology domains and cross-project knowledge
- [[patterns/Patterns|Patterns]] — Universal patterns and conventions
- [[todos/Active TODOs|Active TODOs]] — Pending work items
- [[sessions/|Sessions]] — Session summaries

## Quick Start

1. Run `/obs analyze` in any project to populate the vault
2. The agent auto-orients from this vault at session start
3. Use `/obs note` to capture discoveries as you work
4. Use `/obs lookup` to search across all knowledge
```

**`{vault_path}/projects/Projects.md`:**
```markdown
---
tags: [meta]
type: index
---

# Projects

| Project | Status | Language | Links |
|---------|--------|----------|-------|

_Run `/obs analyze` in a project to add it here._
```

**`{vault_path}/domains/Domains.md`:**
```markdown
---
tags: [meta]
type: index
---

# Domains

| Domain | Projects Using |
|--------|---------------|

_Domains are auto-populated when projects are analyzed._
```

**`{vault_path}/patterns/Patterns.md`:**
```markdown
---
tags: [meta]
type: index
---

# Patterns

| Pattern | Scope | Projects |
|---------|-------|----------|

_Use `/obs note pattern <name>` to add patterns._
```

**`{vault_path}/todos/Active TODOs.md`:**
```markdown
---
tags: [meta, todos]
type: index
---

# Active TODOs

_Group items by project. Use checkboxes for pending, move completed to ## Completed._

## Pending

## In Progress

## Completed
```

**`{vault_path}/templates/project.md`:**
```markdown
---
aliases: []
tags: [project/{short-name}]
type: project
status: active
repo:
path:
language:
framework:
created: {{date}}
---

# {Project Name}

## Architecture

## Components

| Component | Purpose | Key Files |
|-----------|---------|-----------|

## Project Patterns

## Architecture Decisions

## Key Dependencies

## Domains
```

**`{vault_path}/templates/component.md`:**
```markdown
---
tags: [components, project/{short-name}]
type: component
project: "[[projects/{project}/{project}]]"
created: {{date}}
status: active
layer:
depends-on: []
depended-on-by: []
key-files: []
---

# {Component Name}

## Purpose

## Gotchas
```

**`{vault_path}/templates/adr.md`:**
```markdown
---
tags: [architecture, decision, project/{short-name}]
type: adr
project: "[[projects/{project}/{project}]]"
status: proposed
created: {{date}}
---

# ADR-{NNNN}: {Title}

## Context

## Decision

## Consequences

## Alternatives Considered
```

**`{vault_path}/templates/pattern.md`:**
```markdown
---
tags: [patterns, project/{short-name}]
type: pattern
project: "[[projects/{project}/{project}]]"
created: {{date}}
---

# {Pattern Name}

## Problem

## Solution

## When to Use

## Examples
```

**`{vault_path}/templates/session.md`:**
```markdown
---
tags: [sessions]
type: session
projects:
  - "[[projects/{project}/{project}]]"
created: {{date}}
branch:
---

# Session — {{date}}

## Goals

## What Happened

## Decisions Made

## Open Questions

## Next Steps
```

**`{vault_path}/.obsidian/app.json`:**
```json
{
  "alwaysUpdateLinks": true,
  "newFileLocation": "folder",
  "newFileFolderPath": "inbox",
  "attachmentFolderPath": "attachments"
}
```

Announce: `[OBS-SETUP] Vault created at {vault_path}`

---

### Phase 4: Scaffold Current Project

Detect the project name using directory-based detection (git preferred, folder name as fallback):

```bash
basename $(git rev-parse --show-toplevel 2>/dev/null) 2>/dev/null || basename $(pwd)
```

This works in any folder, not just git repos.

Check if `{vault_path}/projects/{name}/{name}.md` already exists.

- If **already scaffolded**: announce `[OBS-SETUP] Project {name} already in vault — skipping scaffold` and skip to Phase 5.
- If **not scaffolded**:

  1. Create directories:
     - `{vault_path}/projects/{name}/`
     - `{vault_path}/projects/{name}/architecture/`
     - `{vault_path}/projects/{name}/components/`
     - `{vault_path}/projects/{name}/patterns/`

  2. Detect what you can from the project directory (check for `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `*.sln`, etc.) to auto-fill language and framework fields.

  3. Try to detect repo URL: `git remote get-url origin`

  4. Write `{vault_path}/projects/{name}/{name}.md` using the project template, substituting:
     - `{Project Name}` → title-cased project name
     - `{short-name}` → `{name}`
     - `repo:` → detected remote URL (or blank)
     - `path:` → absolute path to the working directory
     - `language:` → detected language (or blank)
     - `framework:` → detected framework (or blank)
     - `{{date}}` → today's date in `YYYY-MM-DD`

  5. Create a project-specific `CLAUDE.md` in the working directory (if one doesn't already exist). Read the project's source files to fill in real values:
     ```markdown
     # {Project Name}

     ## Project Context
     {2-3 sentence description based on README or source files}
     Vault overview: `{vault_path}/projects/{name}/{name}.md`

     ## Tech Stack
     - {detected language}
     - {detected framework/dependencies}

     ## Conventions
     - {any conventions detected from config files, linters, etc.}

     ## Key Paths
     - {important directories and files}

     ## When Working on This Project
     - Check vault for existing context before deep-diving into code
     ```
     Keep it under 80 lines. If a `CLAUDE.md` already exists, don't overwrite — just suggest adding a vault reference line if missing.

  Announce: `[OBS-SETUP] Project {name} scaffolded at {vault_path}/projects/{name}/`

---

### Phase 5: Generate CLAUDE.md Snippet

Output the following block with `{vault_path}` replaced by the actual resolved path. **Leave all other placeholders (`{name}`, `{tech}`, `{short-name}`) as-is** — they are generic references in the instructions, not values to substitute:

```markdown
## Obsidian Knowledge Vault

Persistent cross-session, cross-project memory using an Obsidian vault accessed via direct file tools (Read, Write, Edit, Glob, Grep). No MCP server required.

Uses installed skills: `obs-memory` (vault structure + commands), `obsidian-markdown` (proper Obsidian syntax), `obsidian` (best practices).

**Vault path:** `{vault_path}`
Set `$OBSIDIAN_VAULT_PATH` = `{vault_path}` for all vault operations.

### Vault Structure

    {vault_path}/
    ├── Home.md                           # Dashboard
    ├── projects/{name}/
    │   ├── {name}.md                     # Project overview — START HERE
    │   ├── architecture/                 # ADRs and design decisions
    │   ├── components/                   # Per-component notes
    │   └── patterns/                     # Project-specific patterns
    ├── domains/{tech}/                   # Cross-project knowledge
    ├── patterns/                         # Universal patterns
    ├── sessions/                         # Session logs
    ├── todos/Active TODOs.md             # Pending work
    ├── templates/                        # Note templates
    └── inbox/                            # Unsorted

### Session Start — Auto-Orient (max 2 reads)

**On the very first message of any session** (even "hey", "hi", or anything vague), auto-orient immediately:

1. Read `{vault_path}/todos/Active TODOs.md` — know what's pending
2. Detect current project from working directory using `basename $(git rev-parse --show-toplevel 2>/dev/null) 2>/dev/null || basename $(pwd)`. Check if a matching project exists in `{vault_path}/projects/*/`. If found, read `{vault_path}/projects/{name}/{name}.md`. The project overview has wikilinks to components, patterns, ADRs — do NOT follow those links yet; follow on demand.
3. Greet with 2-3 lines of context: what project, what's active/pending. Keep it brief and useful, not ceremonial.

If the user's first message is a specific task, orient AND start working — don't make them wait for a greeting before getting to it.

**No match found — New Project Setup:**
If the working directory doesn't match any project in the vault, this is a new project:
1. Tell the user: "This directory isn't in the vault yet. Want me to set it up?"
2. If yes, run `/obs project` which will:
   a. Create vault project folder and project note
   b. Create a lean `CLAUDE.md` in the project directory (tech stack, conventions, key paths)
   c. Add the project to the vault's Projects.md index
   d. Auto-detect language, framework, and repo URL from project files
3. Then proceed with normal auto-orient using the new project note

### During Work — Lookup Before Reading

- **Frontmatter first:** When scanning notes, read first ~10 lines to check `tags`, `type`, `status`, `project` before reading the full body.
- **List before read:** Glob directory contents before reading individual files.
- **Follow links on demand:** The project overview wikilinks to components/patterns/ADRs. Only read those when the current task needs them.
- **Search:** Use `Grep` across `{vault_path}/` for freetext search across the vault.

### Writing to the Vault

Use Obsidian-flavored markdown (wikilinks `[[Note]]`, frontmatter properties, callouts `> [!type]`). Follow the `obsidian-markdown` skill syntax.

**When to write:**
- New component deeply analyzed → create component note
- Architecture decision made → create ADR
- Pattern identified → create pattern note
- Session ending → write session summary (if user requests — not automatic)

**Frontmatter conventions (always include):**

    ---
    tags: [category, project/short-name]
    type: <component|adr|session|project|pattern>
    project: "[[projects/{name}/{name}]]"
    created: YYYY-MM-DD
    ---

**Scoping:**
| Knowledge type | Location |
|---|---|
| One project only | `projects/{name}/` |
| Shared across projects | `domains/{tech}/` |
| Universal, tech-agnostic | `patterns/` |
| Session summaries | `sessions/` |
| TODOs | `todos/Active TODOs.md` |

**Write concisely:** Bullet points over prose. Wikilinks over repeated explanations. Tags for discoverability.

### Command Routing

When the user types `/obs <command>`, invoke the obs-vault plugin's obs-memory skill:

| Command | Action |
|---|---|
| `/obs analyze` | Deep-analyze current project, populate vault |
| `/obs recap` | Write session summary (manual — ask for it when you want one) |
| `/obs project [name]` | Scaffold new project |
| `/obs note <type> [name]` | Create component/adr/pattern/session note |
| `/obs todo [action]` | Manage TODOs |
| `/obs init [path]` | Create a new vault |
| `/obs lookup <query>` | Explicit vault search (usually not needed — agent searches naturally) |
| `/obs relate <src> <tgt>` | Create relationships between notes |

### Rules

- Access vault via Read/Write/Edit/Glob/Grep — no MCP needed
- Never store source code in the vault — only context, decisions, summaries
- Session logs under 50 lines
- The vault is the single source of truth for cross-session context
- Use Obsidian-flavored markdown (wikilinks, frontmatter, callouts)
```

---

### Phase 6: Instructions

Tell the user:

1. **Add to CLAUDE.md:** "Copy the snippet above into your `~/.claude/CLAUDE.md`. If you already have an Obsidian vault section, replace it — don't duplicate."
2. **Open in Obsidian (optional):** "Vault Switcher → Open folder as vault → `{vault_path}`"
3. **Populate the vault:** "Run `/obs analyze` in any project to populate it with project knowledge."

---

### Phase 7: Summary

Report what was done:

```
[OBS-SETUP COMPLETE]

Vault: {vault_path} ({created/already existed})
Project: {name} ({scaffolded/already scaffolded/not scaffolded})
CLAUDE.md snippet: generated above

Next steps:
1. Add the snippet to ~/.claude/CLAUDE.md
2. Run `/obs analyze` to populate vault with project knowledge
3. (Optional) Open {vault_path} in Obsidian as a vault
```

---

## Error Recovery

| Error | Action |
|-------|--------|
| Cannot resolve `~` | Use `$HOME/Documents/AgentMemory` as fallback (or `$TEMP/AgentMemory` on Windows), warn user |
| Directory creation fails (permissions) | Report the specific path and error, stop |
| Git not installed | Fall back to `basename $(pwd)` for project name, note in summary |
| `git remote get-url origin` fails | Leave `repo:` blank in project template |
| Project template write fails | Warn, skip scaffold, continue to Phase 5 |
| Vault already fully set up | Skip all creation, still output Phase 5 snippet and Phase 6 instructions |
