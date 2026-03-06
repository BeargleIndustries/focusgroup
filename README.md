# Beargle Industries Claude Code Plugins

A growing collection of Claude Code plugins by [Beargle Industries](https://github.com/BeargleIndustries). New plugins are actively in development — add the marketplace once and get access to everything as it ships.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [focusgroup](plugins/focusgroup/) | AI-powered focus group reviews with deep protocols, advanced methodologies, quality control, and rich output formats |
| [changelog](plugins/changelog/) | Changelog generation with doc-vs-code consistency auditing — catches wrong install instructions, outdated API examples, nonexistent CLI flags, and more |
| *More coming soon* | |

## Install

Add this marketplace to Claude Code:

```bash
claude plugin marketplace add BeargleIndustries/beargle-plugins
```

Then install any plugin:

```bash
claude plugin install focusgroup
claude plugin install changelog
```

Restart Claude Code after installing.

## Update

If new plugins have been added to the marketplace, update the marketplace cache first:

```bash
claude plugin marketplace update beargle-plugins
```

Then install or update individual plugins:

```bash
claude plugin install changelog
claude plugin update focusgroup
```

## License

[MIT](LICENSE)
