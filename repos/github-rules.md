# Rules: synman/github-rules

Rules specific to the **synman/github-rules** repository.  These rules extend the global rules in `GLOBAL.md` and apply when working in this repository.

## 1. Purpose of this repository

This repository is documentation-only — it contains no application code.  All changes must be Markdown files.  Do not add source code, scripts, binaries, or build tooling here.

## 2. Keep rules concise and actionable

Each rule in `GLOBAL.md` and in any `repos/*.md` file should be short (a few sentences), practical, and clearly actionable.  Avoid vague guidance.

## 3. Follow the established file format

- One `##` section per rule, numbered sequentially.
- Global rules go in `GLOBAL.md`.
- Repo-specific rules go in `repos/<repo-name>.md`, using `repos/_template.md` as the starting point.
- Do not add rules directly to `README.md` or `.github/copilot-instructions.md`; keep those as structural/navigational files.

## 4. Update `.github/copilot-instructions.md` when global rules change

After merging any change to `GLOBAL.md`, check whether `.github/copilot-instructions.md` needs to be updated to keep its summary in sync.

## 5. No PRs for questions about this repository's purpose

If someone asks a general question about how this rules system works, answer it directly in chat.  Do not open a PR to document the answer.
