# Copilot Instructions for synman/github-rules

This repository is the authoritative source for Copilot behavioral rules across all workspace projects.

## Structure

- **[`global/copilot-instructions.md`](../global/copilot-instructions.md)** — universal rules that apply to every repository and session.
- **[`projects/<repo-name>/copilot-instructions.md`](../projects/)** — per-project extensions and overrides. Repo-specific rules take precedence over global rules when they conflict.

## Projects tracked

| Project | Rules file |
|---------|-----------|
| bambu-printer-app | [`projects/bambu-printer-app/copilot-instructions.md`](../projects/bambu-printer-app/copilot-instructions.md) |
| bambu-printer-manager | [`projects/bambu-printer-manager/copilot-instructions.md`](../projects/bambu-printer-manager/copilot-instructions.md) |
| bambu-mcp | [`projects/bambu-mcp/copilot-instructions.md`](../projects/bambu-mcp/copilot-instructions.md) |
| bambu-fw-fetch | [`projects/bambu-fw-fetch/copilot-instructions.md`](../projects/bambu-fw-fetch/copilot-instructions.md) |
| bambu-mqtt | [`projects/bambu-mqtt/copilot-instructions.md`](../projects/bambu-mqtt/copilot-instructions.md) |
| webcamd | [`projects/webcamd/copilot-instructions.md`](../projects/webcamd/copilot-instructions.md) |

## Rules for this repository

- This repo is **documentation-only**. No code, no build artifacts.
- Changes to `global/copilot-instructions.md` must also be applied to `~/.copilot/copilot-instructions.md` on the local machine (the live rules file).
- Changes to `projects/<repo>/copilot-instructions.md` must also be applied to `~/<repo>/.github/copilot-instructions.md` on the local machine.
- Keep rule files concise and actionable. Avoid prose that could be expressed as a bullet or table.
- All changes via pull request — no direct pushes to `main`.
