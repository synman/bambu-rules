# github-rules

A central place to store global rules and per-repo overrides for Copilot and other coding assistants.

## Purpose

This repository provides a single source of truth for behavioral guidelines that Copilot and similar assistants should follow when working across all repositories owned by this account.  Two layers of rules are supported:

- **Global rules** (`GLOBAL.md`) — apply to every repository unless overridden.
- **Repo-specific rules** (`repos/<repo-name>.md`) — apply only to the named repository.

## Structure

```
github-rules/
├── .github/
│   └── copilot-instructions.md      # Wires rules into GitHub Copilot for this repo
├── GLOBAL.md                        # Global rules for all repos
└── repos/
    ├── _template.md                 # Template for new repo rule files
    ├── bambu-printer-manager.md     # Rules specific to synman/bambu-printer-manager
    └── github-rules.md              # Rules specific to synman/github-rules (this repo)
```

## Precedence

When a rule in a repo-specific file conflicts with a rule in `GLOBAL.md`, the **repo-specific rule wins**.  All non-conflicting global rules still apply.

## How to add rules for a new repository

1. Copy `repos/_template.md` to `repos/<repo-name>.md` (use the exact GitHub repository name, lowercased).
2. Fill in the repo-specific sections — remove any sections that are not relevant.
3. Open a pull request against the default branch of this repository.
4. Once merged, the new rules take effect immediately for that repository.
