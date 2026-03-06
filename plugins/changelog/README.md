# changelog

A Claude Code plugin that generates changelogs from git history and audits documentation against the actual codebase to catch inconsistencies.

AI coding agents are prolific documentation writers — and they frequently write docs that look plausible but won't work. Wrong install commands, nonexistent CLI flags, outdated API signatures, broken file paths. This plugin catches that drift before your users do.

## Install

```bash
claude plugin marketplace add BeargleIndustries/beargle-plugins
claude plugin install changelog
```

Restart Claude Code after installing.

## Commands

- `/changelog` — generate changelog + run doc audit (default)
- `/changelog --audit` — doc audit only
- `/changelog --since v1.0.0` — changelog since a specific tag
- `/changelog --fix` — auto-fix unambiguous doc issues
- `/doccheck` (or `/doc-check`) — alias for `/changelog --audit`

## What It Catches

- Install instructions that reference nonexistent packages or wrong commands
- API docs with wrong function signatures, parameter names, or return types
- CLI docs listing flags that don't exist in the argument parser
- Code examples using renamed or removed APIs
- Config docs with keys the code doesn't read
- File paths that don't exist
- Documented defaults that differ from actual code defaults
- Code changes without corresponding doc updates
- Docs still referencing removed features

## Output

Results go to `.changelog/{timestamp}/` with:
- A unified report with documentation health score (0-100)
- Per-category audit details
- Generated changelog (keep-a-changelog format)
- Cross-reference of code changes vs doc updates
- Auto-fix log (when using `--fix`)

## Options

| Flag | Description |
|------|-------------|
| `--since TAG` | Changelog starting point |
| `--audit` | Doc audit only |
| `--full` | Comprehensive deep audit + changelog (more thorough than default) |
| `--fix` | Auto-fix unambiguous issues |
| `--format md\|json` | Output format |
| `--out <dir>` | Custom output directory |
| `--scope <path>` | Limit to subdirectory |
| `--audience dev\|user\|both` | Changelog audience |
| `--group-by type\|scope\|component` | Changelog grouping |

## License

[MIT](../../LICENSE)
