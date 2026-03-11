---
name: obs-memory
description: "Persistent Obsidian-based memory for Claude Code. Orients at session start, captures discoveries during work, and maintains context across sessions. Activate when the user mentions obsidian memory, obsidian vault, obsidian notes, or /obs commands. Commands: init, analyze, recap, project, note, todo, lookup, relate."
metadata:
  author: beargleindustries
  version: "2.2"
license: MIT
---

# Obsidian Agent Memory

You have access to a persistent Obsidian knowledge vault — a graph-structured memory that persists across sessions. Use it to orient yourself, look up architecture and component knowledge, and write back discoveries.

## Vault Discovery

Resolve the vault path using this chain (first match wins):

1. **Environment variable**: `$OBSIDIAN_VAULT_PATH`
2. **Agent config reference**: Parse the vault path from the agent's project or global config (look for "Obsidian Knowledge Vault" section with a path like `~/Documents/SomeName/`)
3. **Default**: `~/Documents/AgentMemory`

Store the resolved path as `$VAULT` for all subsequent operations. Derive `$VAULT_NAME` as `basename "$VAULT"` for CLI calls.

Verify the vault exists by checking for `$VAULT/Home.md`. If the vault doesn't exist, inform the user and suggest running the `init` command to bootstrap a new vault.

## Session Start — Orientation

At the start of every session, orient yourself with **at most 2 operations**:

### Step 1: Read TODOs

Read `$VAULT/todos/Active TODOs.md` directly.

Know what's pending, in-progress, and recently completed.

### Step 2: Detect current project and read its overview

Auto-detect the project from the current working directory (git repo name preferred, folder name as fallback):
```bash
basename $(git rev-parse --show-toplevel 2>/dev/null) 2>/dev/null || basename $(pwd)
```

Then check if a matching project exists by listing files in `$VAULT/projects/*/`. Match the working directory name (git repo preferred, folder name as fallback) against project folder names. If a match is found, read the project overview at `$VAULT/projects/{matched-name}/{matched-name}.md`.

This project overview contains wikilinks to all components, patterns, architecture decisions, and domains. **Do not read those linked notes yet** — follow them on demand when the current task requires that context.

### What NOT to read at session start
- `Home.md` (only if you're lost and can't find the project)
- `sessions/` (only if the user references prior work)
- Domain indexes (only if you need cross-project knowledge)
- Component notes (only when working on that component)

## Automatic Behaviors

These behaviors apply to any agent using this skill. They do not require explicit commands.

### On session start

Auto-orient (TODOs + project overview) without being asked, following the Session Start procedure above. If the vault doesn't exist at the resolved path, inform the user and suggest running `init`.

### On session end signals

When the user says "done", "wrapping up", "that's it", "let's stop", or similar end-of-session language — offer to write a session summary. Don't auto-run; ask first: "Want me to write a session summary to the vault before we wrap up?"

### On component discovery

When you deeply analyze a component that has no vault note — and the project has an active vault — offer to create a component note and infer relationships from imports and dependencies. Example: "I noticed there's no vault note for the AuthMiddleware component. Want me to create one and map its dependencies?"

### On first run

When the vault doesn't exist at any resolved path, guide the user through `init`, then auto-scaffold the current project if inside a git repo.

## During Work — Graph Navigation

**Principle: Use file tools (Read, Glob, Grep) as the primary method.** The Obsidian CLI is not typically available; file tools are always available.

### File-based lookups (primary)

- Need to understand a component? The project overview links to it. Read that one note.
- Need an architecture decision? The component note or project overview links to it. Follow the link.
- Need cross-project knowledge? Component/pattern notes link to domain notes. Follow the link.
- Need session history? Only read if you're stuck or the user references prior work.
- Need to search? Use Grep across `$VAULT/**/*.md`.

### Obsidian CLI (optional — only if installed)

If the Obsidian CLI is available, you can use it for targeted queries:
```bash
obsidian vault=$VAULT_NAME property:read file="Component Name" name="depends-on"
obsidian vault=$VAULT_NAME backlinks file="Component Name"
obsidian vault=$VAULT_NAME search format=json query="search term" matches limit=10
```
Where `$VAULT_NAME` is `basename "$VAULT"`. Do not assume the CLI is present; verify first.

### Frontmatter-first scanning
When you need to scan multiple notes to find the right one, read just the first ~10 lines of each file. The `tags`, `project`, `type`, and `status` fields in the frontmatter tell you if the note is relevant before reading the full body.

### Directory listing before reading
List directory contents before reading files — know what exists without consuming tokens:
- `$VAULT/projects/{name}/**/*.md` — all notes for a project
- `$VAULT/domains/{tech}/*.md` — domain knowledge files

## Writing to the Vault

Write concisely. Notes are for your future context, not human documentation. Prefer:
- Bullet points over prose
- Wikilinks over repeated explanations (link to it, don't re-state it)
- Frontmatter tags for discoverability over verbose descriptions

### When to write
- **New component discovered**: Create a component note when you deeply understand a part of the codebase
- **Architecture decision made**: Record ADRs when significant design choices are made
- **Pattern identified**: Document recurring patterns that future sessions should follow
- **Domain knowledge learned**: Write to domain notes when you discover cross-project knowledge

### Scoping rules
| Knowledge type | Location | Example |
|---|---|---|
| One project only | `projects/{name}/` | How this API handles auth |
| Shared across projects | `domains/{tech}/` | How Go interfaces work |
| Universal, tech-agnostic | `patterns/` | SOLID principles |
| Session summaries | `sessions/` | What was done and discovered |
| TODOs | `todos/Active TODOs.md` | Grouped by project |

### Frontmatter conventions
Always include in new notes:
```yaml
---
tags: [category, project/short-name]
type: <component|adr|session|project>
project: "[[projects/{name}/{name}]]"
created: YYYY-MM-DD
---
```

### Wikilink conventions
- Link to related notes: `[[projects/{name}/components/Component Name|Component Name]]`
- Link to domains: `[[domains/{tech}/{Tech Name}|Tech Name]]`
- Link back to project: `[[projects/{name}/{name}|project-name]]`

### Note templates

**Component Note:**
```yaml
---
tags: [components, project/{short-name}]
type: component
project: "[[projects/{name}/{name}]]"
created: {date}
status: active
layer: ""
depends-on: []
depended-on-by: []
key-files: []
---
```
Sections: Purpose, Gotchas

**Architecture Decision:**
```yaml
---
tags: [architecture, decision, project/{short-name}]
type: adr
project: "[[projects/{name}/{name}]]"
status: proposed | accepted | superseded
created: {date}
---
```
Sections: Context, Decision, Alternatives Considered, Consequences

**Session Note:**
```yaml
---
tags: [sessions]
type: session
projects:
  - "[[projects/{name}/{name}]]"
created: {date}
branch: {branch-name}
---
```
Sections: Context, Work Done, Discoveries, Decisions, Next Steps

## Commands

### `init` — Initialize the Vault

Bootstrap a new Obsidian Agent Memory vault by creating directories and starter files directly.

**Usage**: `init [path]`

#### Steps:

1. **Determine vault path**: Use the first argument if provided, otherwise use the vault resolution chain (default: `~/Documents/AgentMemory`).

2. **Check if vault already exists**: Look for `$VAULT/Home.md`. If it exists, tell the user the vault already exists at that path and offer to open it.

3. **Create directory structure**:
   ```bash
   mkdir -p "$VAULT"
   mkdir -p "$VAULT/projects"
   mkdir -p "$VAULT/domains"
   mkdir -p "$VAULT/patterns"
   mkdir -p "$VAULT/sessions"
   mkdir -p "$VAULT/todos"
   mkdir -p "$VAULT/templates"
   mkdir -p "$VAULT/inbox"
   mkdir -p "$VAULT/attachments"
   mkdir -p "$VAULT/.obsidian"
   ```

4. **Write starter files** directly using file tools:

   **`$VAULT/Home.md`**:
   ```markdown
   ---
   tags: [home]
   type: dashboard
   created: {YYYY-MM-DD}
   ---
   # Agent Memory Vault

   ## Active Projects
   See [[projects/Projects|Projects Index]]

   ## Pending Work
   See [[todos/Active TODOs|Active TODOs]]

   ## Recent Sessions
   See [[sessions/Session Log|Session Log]]

   ## Domains
   See [[domains/Domains|Domains Index]]
   ```

   **`$VAULT/projects/Projects.md`**:
   ```markdown
   ---
   tags: [index, projects]
   type: index
   created: {YYYY-MM-DD}
   ---
   # Projects

   | Project | Language | Framework | Status |
   |---------|----------|-----------|--------|
   ```

   **`$VAULT/todos/Active TODOs.md`**:
   ```markdown
   ---
   tags: [todos]
   type: todos
   created: {YYYY-MM-DD}
   ---
   # Active TODOs

   ## Pending

   ## In Progress

   ## Completed
   ```

   **`$VAULT/sessions/Session Log.md`**:
   ```markdown
   ---
   tags: [sessions, log]
   type: log
   created: {YYYY-MM-DD}
   ---
   # Session Log

   | Date | Project | Branch | Summary |
   |------|---------|--------|---------|
   ```

   **`$VAULT/domains/Domains.md`**:
   ```markdown
   ---
   tags: [index, domains]
   type: index
   created: {YYYY-MM-DD}
   ---
   # Domains

   | Domain | Category | Projects |
   |--------|----------|----------|
   ```

5. **Write Obsidian config** to `$VAULT/.obsidian/app.json`:
   ```json
   {
     "alwaysUpdateLinks": true,
     "newFileLocation": "folder",
     "newFileFolderPath": "inbox",
     "attachmentFolderPath": "attachments"
   }
   ```

6. **Create `.gitkeep` files** in `inbox/` and `attachments/`.

7. **Report** the created vault and provide next steps:
   - Open in Obsidian: Vault Switcher → Open folder as vault → `$VAULT`
   - Set the vault path via `OBSIDIAN_VAULT_PATH` environment variable or agent config
   - Start working — the agent will build the knowledge graph as it goes

8. **Generate agent config snippet**: Output a vault path snippet for the user's agent. For Claude Code, output a `CLAUDE.md` snippet:
   ```markdown
   ## Obsidian Knowledge Vault
   Persistent knowledge vault at `$VAULT`.
   Set `$OBSIDIAN_VAULT_PATH` = `$VAULT` for all vault operations.
   ```
   For other agents, output: "Add `OBSIDIAN_VAULT_PATH=$VAULT` to your environment or agent config."

9. **Auto-scaffold current project**: If inside a git repo, automatically run the `project` command to scaffold the current project in the vault.

10. **Concise output**: Keep the final output to 5-8 lines max: vault path created, project scaffolded (if applicable), how to open in Obsidian, how to set the vault path.

### `analyze` — Analyze Project & Hydrate Vault

Analyze the current codebase and populate the vault with interconnected, content-rich notes.

**Usage**: `analyze` (no arguments — uses current repo)

This command delegates deep analysis to a sub-agent:

```
Agent(subagent_type="obs-vault:vault-analyst")
```

The vault-analyst sub-agent handles all four phases below. Pass it the resolved `$VAULT` path and the detected project name.

#### Phase 1: Discovery — Scan for Knowledge Sources

Scan the repo for files that contain pre-existing knowledge:

| Category | Files to scan |
|---|---|
| Agent configs | `CLAUDE.md`, `.claude/CLAUDE.md`, `.cursorrules`, `.windsurfrules`, `.clinerules`, `AGENTS.md`, `Agents.md` |
| Documentation | `README.md`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, `docs/architecture.md`, `docs/ARCHITECTURE.md` |
| Existing ADRs | `docs/adr/ADR-*.md`, `architecture/ADR-*.md`, `adr/*.md`, `docs/decisions/*.md` |
| Project metadata | `package.json`, `go.mod`, `Cargo.toml`, `pyproject.toml`, `setup.py`, `Gemfile`, `pom.xml`, `build.gradle`, `*.csproj` |
| Build/CI | `Makefile`, `Dockerfile`, `docker-compose.yml`, `.github/workflows/*.yml`, `.gitlab-ci.yml` |
| Config | `tsconfig.json`, `.eslintrc.*`, `jest.config.*`, `.goreleaser.yml` |

Read each discovered file. For large files (README, agent configs), read fully. For metadata files, extract key fields (name, version, dependencies).

Also gather:
- Repo URL from `git remote get-url origin`
- Repo root path from `git rev-parse --show-toplevel`
- Active branch from `git branch --show-current`
- Directory tree (top 2 levels of source directories, excluding hidden/vendor/node_modules)
- File extension frequency (for language detection)

#### Phase 2: Analysis — Extract & Synthesize

Using the discovered content, synthesize:

1. **Project metadata**: name, language(s), framework(s), repo URL, local path
2. **Architecture summary**: Entry points, layer organization, build system
3. **Component inventory**: Major functional modules — each top-level source directory or logical grouping. For each: purpose, key files, and relationships
4. **Pattern inventory**: Coding conventions, error handling strategies, testing approaches — extracted from agent config files
5. **Domain mapping**: Detected technologies → vault domain notes
6. **Existing decisions**: ADR files found in the repo → import as vault ADR notes
7. **Dependency summary**: Key dependencies from package manifests

#### Phase 3: Hydration — Write Vault Notes

**Idempotency rules:**
- If project directory doesn't exist → create everything (scaffold + populate)
- If project directory exists but overview is a skeleton → **replace** overview with populated version
- If individual component/pattern/ADR notes already exist → **skip** and report (don't overwrite manual work)
- Domain notes: create if missing, **append** project link if existing

**Notes to write:**

1. **Project overview** (`$VAULT/projects/{name}/{name}.md`) — Fully populated:
   ```yaml
   ---
   aliases: []
   tags: [project/{short-name}]
   type: project
   repo: {git remote url}
   path: {repo root path}
   language: {detected language(s)}
   framework: {detected framework(s)}
   created: {YYYY-MM-DD}
   status: active
   ---
   ```
   Sections:
   - **Architecture**: Real description from analysis
   - **Components**: Table with wikilinks to component notes
   - **Project Patterns**: Table with wikilinks to pattern notes
   - **Architecture Decisions**: List with wikilinks to ADR notes
   - **Key Dependencies**: From package manifests
   - **Domains**: Wikilinks to domain notes

2. **Component notes** (`$VAULT/projects/{name}/components/{Component}.md`) — One per major module

3. **Pattern notes** (`$VAULT/projects/{name}/patterns/{Pattern}.md`) — From agent config conventions

4. **ADR imports** (`$VAULT/projects/{name}/architecture/ADR-{NNNN} {title}.md`) — From existing repo ADRs

5. **Domain notes** (`$VAULT/domains/{tech}/{Tech}.md`):
   - If new: create with project link
   - If existing: add this project to "Projects Using This Domain" section

6. **Index updates**:
   - `$VAULT/projects/Projects.md` — add/update row
   - `$VAULT/domains/Domains.md` — add/update rows for new domains

#### Phase 4: Report

Print a summary:
```
Analyzed: {project-name}
  Sources read: {N} knowledge files
  Created: project overview (populated)
  Created: {N} component notes
  Created: {N} pattern notes
  Imported: {N} architecture decisions
  Linked: {N} domain notes
  Skipped: {N} existing notes (preserved)
```

### `recap` — Write Session Summary

When the user asks to save a session summary, or runs `/obs recap`, write a session summary note and update TODOs.

**Usage**: `recap`

#### Steps:

1. **Gather session context** by running:
   ```bash
   git log --oneline -20
   git diff --stat HEAD~5..HEAD 2>/dev/null || git diff --stat
   git branch --show-current
   ```

2. **Read current TODOs** from `$VAULT/todos/Active TODOs.md`.

3. **Read project overview** from `$VAULT/projects/$PROJECT/$PROJECT.md` (for wikilinks and context).

4. **Write session note** directly at `$VAULT/sessions/{YYYY-MM-DD} - {title}.md`:
   ```yaml
   ---
   tags: [sessions]
   type: session
   projects:
     - "[[projects/$PROJECT/$PROJECT]]"
   created: {YYYY-MM-DD}
   branch: {current-branch}
   ---
   ```
   Sections to fill:
   - **Context**: What was being worked on (from git log context)
   - **Work Done**: Numbered list of accomplishments (from commits and diffs)
   - **Discoveries**: Technical findings worth remembering
   - **Decisions**: Design choices made during this session
   - **Next Steps**: What should happen next (checkboxes)

5. **Update TODOs**: Edit `$VAULT/todos/Active TODOs.md`:
   - Move completed items to Completed section
   - Add new items discovered during the session
   - Keep items grouped by project

6. **Update Session Log**: Add an entry to `$VAULT/sessions/Session Log.md` with the date, project, branch, and a one-line summary.

7. **Report** what was written.

### `project` — Scaffold New Project

Scaffold a new project in the vault. Uses the first argument as the project name, or defaults to `$PROJECT`.

**Usage**: `project [name]`

#### Steps:

1. **Determine project name**: Use the argument if provided, otherwise use the working directory name (git repo preferred, folder name as fallback):
   ```bash
   basename $(git rev-parse --show-toplevel 2>/dev/null) 2>/dev/null || basename $(pwd)
   ```

2. **Check if project exists**: Look for `$VAULT/projects/{name}/{name}.md`. If it exists, tell the user and offer to open it instead.

3. **Create directory structure**:
   - `$VAULT/projects/{name}/`
   - `$VAULT/projects/{name}/architecture/`
   - `$VAULT/projects/{name}/components/`
   - `$VAULT/projects/{name}/patterns/`

4. **Create project overview** at `$VAULT/projects/{name}/{name}.md`:
   ```yaml
   ---
   aliases: []
   tags: [project/{short-name}]
   type: project
   repo: {git remote url if available}
   path: {working directory}
   language: {detected from files}
   framework:
   created: {YYYY-MM-DD}
   status: active
   ---
   ```
   Sections: Architecture, Components, Project Patterns, Architecture Decisions, Domains

   Auto-detect and fill:
   - Language from file extensions in the repo
   - Repo URL from `git remote get-url origin`
   - Link to relevant domains that exist in `$VAULT/domains/`

5. **Update Projects.md**: Add a row to the project table in `$VAULT/projects/Projects.md`.

6. **Create project-specific CLAUDE.md** in the working directory (if one doesn't already exist):

   Write a lean `CLAUDE.md` at the project root with this structure:
   ```markdown
   # {Project Name}

   ## Project Context
   {2-3 sentence description based on README, package.json, or source files}
   Vault overview: `{vault_path}/projects/{name}/{name}.md`

   ## Tech Stack
   - {detected language}
   - {detected framework/dependencies}

   ## Conventions
   - {any conventions detected from config files, linters, etc.}

   ## Key Paths
   - {important directories and files detected from project structure}

   ## When Working on This Project
   - Check vault for existing context before deep-diving into code
   ```

   Read the project's source (README, package.json/pyproject.toml/Cargo.toml, config files) to fill in real values — don't leave placeholders. Keep it under 80 lines. This file gives Claude Code per-project grounding without bloating the global config.

   If a `CLAUDE.md` already exists in the working directory, do NOT overwrite it. Instead, check if it has a vault reference line. If not, suggest the user add: `Vault overview: {vault_path}/projects/{name}/{name}.md`

7. **Report** the scaffolded structure.

### `note` — Create a Note from Template

Create a note using a template. The first argument specifies the type: `component`, `adr`, or `pattern`.

**Usage**: `note <component|adr|pattern> [name]`

#### `note component [name]`

Create at `$VAULT/projects/$PROJECT/components/{name}.md`:
```yaml
---
tags: [components, project/{short-name}]
type: component
project: "[[projects/$PROJECT/$PROJECT]]"
created: {YYYY-MM-DD}
status: active
layer: ""
depends-on: []
depended-on-by: []
key-files: []
---
```
Sections: Purpose, Gotchas

If a name argument is provided, use it as the component name. Otherwise, ask the user.

#### `note adr [title]`

Determine the next ADR number by listing existing ADRs in `$VAULT/projects/$PROJECT/architecture/ADR-*.md`.

Create at `$VAULT/projects/$PROJECT/architecture/ADR-{NNNN} {title}.md`:
```yaml
---
tags: [architecture, decision, project/{short-name}]
type: adr
project: "[[projects/$PROJECT/$PROJECT]]"
status: proposed
created: {YYYY-MM-DD}
---
```
Sections: Context, Decision, Alternatives Considered, Consequences

#### `note pattern [name]`

Create at `$VAULT/projects/$PROJECT/patterns/{name}.md`:
```yaml
---
tags: [patterns, project/{short-name}]
project: "[[projects/$PROJECT/$PROJECT]]"
created: {YYYY-MM-DD}
---
```
Sections: Pattern, When to Use, Implementation, Examples

After creating any note, add a wikilink to it from the project overview.

### `todo` — Manage TODOs

View and update the Active TODOs for the current project.

**Usage**: `todo [action]`

#### Steps:

1. **Read current TODOs** from `$VAULT/todos/Active TODOs.md`.

2. **If no additional arguments**: Display the current TODOs for `$PROJECT` and ask what to update.

3. **If arguments provided**: Parse as a TODO action:
   - Plain text → Add as a new pending item under `$PROJECT`
   - `add <text>` or `add: <text>` → Same as plain text (add as pending item)
   - `done <text>` or `done: <text>` → Move matching item to Completed
   - `remove <text>` or `remove: <text>` → Remove matching item

4. **Write back** the updated file.

### `lookup` — Search the Vault

Search the vault for knowledge using Grep across all `.md` files in `$VAULT`.

**Usage**: `lookup <query>`

#### Steps:

1. **Search** for the query term across all `.md` files in `$VAULT` using Grep.
2. **Collect matching files** — for each match, read the first ~10 lines to get the frontmatter (`tags`, `type`, `status`, `project`).
3. **Present results** with file path + frontmatter summary so the user can decide which notes to read in full.

For tag-based searches (query starts with `#` or `project/`), Grep for the tag string in frontmatter. For note-name searches, also Grep for `[[query` to find backlinks.

If the Obsidian CLI is installed, you can also use:
```bash
obsidian vault=$VAULT_NAME search format=json query="<query>" matches limit=10
```

### `relate` — Link Notes

Add wikilinks between notes to express relationships.

**Usage**: `relate <source> <target>`

#### Steps:

1. **Read the source note** at its vault path.
2. **Add a wikilink** to the target in an appropriate section (e.g. under "Related" or inline where relevant). Use `[[path/to/Target|Target]]` format.
3. **Optionally add a backlink** in the target note pointing back to the source. Read the target, add the wikilink in a suitable location.
4. **Report** both notes updated with the links added.

If the Obsidian CLI is installed, you can also manage structured frontmatter relationships:
```bash
obsidian vault=$VAULT_NAME property:set file="<source>" name="depends-on" value="[[Target]]" type="list"
```

#### `relate show <name>`

Display all relationships for a note. Read the note and extract all wikilinks from both frontmatter and body. Grep for `[[<name>` across `$VAULT` to find backlinks. Present both outgoing links and backlinks grouped clearly.

## Token Budget Rules

1. **File tools first**: Use Read, Glob, Grep — always available, no CLI required
2. **Session start**: At most 2 operations (TODOs + project overview)
3. **During work**: Use `lookup` and `relate show` before reading full notes; Grep before reading files
4. **Frontmatter first**: When scanning, read ~10 lines before committing to full read
5. **List before read**: Glob directory contents before reading files
6. **Write concisely**: Bullet points, links, tags — no prose when bullets suffice

## Error Handling

- If the vault doesn't exist → suggest running `/obs init` to bootstrap it
- If the project doesn't exist in the vault → offer to run `/obs project` to scaffold it
- If a note already exists → show it instead of overwriting, offer to edit
- If no git repo is detected → use current directory name as project name
- If CLI command fails → fall back to file read for the same data

## Vault Structure Reference
```
$VAULT/
├── Home.md                           # Dashboard (read only if lost)
├── projects/{name}/
│   ├── {name}.md                     # Project overview — START HERE
│   ├── architecture/                 # ADRs and design decisions
│   ├── components/                   # Per-component notes
│   └── patterns/                     # Project-specific patterns
├── domains/{tech}/                   # Cross-project knowledge
├── patterns/                         # Universal patterns
├── sessions/                         # Session logs (read only when needed)
├── todos/Active TODOs.md             # Pending work (read at session start)
├── templates/                        # Note templates
└── inbox/                            # Unsorted
```
