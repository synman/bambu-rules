# Copilot Instructions

This file wires the rules stored in this repository into GitHub Copilot.

## How rules are structured

- **[`GLOBAL.md`](../GLOBAL.md)** — baseline rules that apply to every repository.
- **[`repos/<repo-name>.md`](../repos/)** — overrides and additions for a specific repository.  When a repo-specific rule conflicts with a global rule, the repo-specific rule takes precedence.

## Rules that always apply (from GLOBAL.md)

1. **Confirm target repo and branch** before opening any pull request.
2. **Answer non-code questions directly** in chat — do not open a PR to document an answer.
3. **Never commit secrets** (API keys, tokens, passwords).  If a secret is exposed, rotate it immediately.
4. **Clarify intent** before performing any write action (push, PR creation, branch deletion, force push).
5. **Mobile-first** when changes touch UI or UX surfaces.

## Rules specific to this repository (synman/github-rules)

See [`repos/github-rules.md`](../repos/github-rules.md).

## Keeping instructions up to date

Whenever `GLOBAL.md` or any file under `repos/` is changed, update the summary above if the change affects the instructions Copilot should follow.
