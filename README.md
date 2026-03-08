# github-rules

Authoritative source for Copilot behavioral rules across all workspace projects.

## Structure

```
github-rules/
├── .github/
│   └── copilot-instructions.md         # Copilot hook for this repo itself
├── global/
│   └── copilot-instructions.md         # Universal rules — all repos, all sessions
└── projects/
    ├── bambu-printer-app/
    │   └── copilot-instructions.md
    ├── bambu-printer-manager/
    │   └── copilot-instructions.md
    ├── bambu-mcp/
    │   └── copilot-instructions.md
    ├── bambu-fw-fetch/
    │   └── copilot-instructions.md
    ├── bambu-mqtt/
    │   └── copilot-instructions.md
    └── webcamd/
        └── copilot-instructions.md
```

## Precedence

Repo-specific rules in `projects/<repo>/` extend and override global rules. All non-conflicting global rules remain in effect.

## Local sync

The live rules files on the local machine are:

| Source (this repo) | Local path |
|--------------------|-----------|
| `global/copilot-instructions.md` | `~/.copilot/copilot-instructions.md` |
| `projects/<repo>/copilot-instructions.md` | `~/<repo>/.github/copilot-instructions.md` |

Changes in this repo must be manually synced to local paths (or vice versa) to take effect in active sessions.

## Adding a new project

1. Create `projects/<repo-name>/copilot-instructions.md` with the project-specific rules.
2. Copy it to `~/<repo-name>/.github/copilot-instructions.md` on the local machine.
3. Add the project to the table in `.github/copilot-instructions.md`.
4. Open a pull request against `main`.

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
