# Global Rules

These rules apply to **all repositories** unless a repo-specific file in `repos/` overrides them.

## 1. Confirm target repo and branch before opening PRs

Always verify the correct repository and target branch with the user before creating or opening a pull request.  Do not assume the default branch or the current working repository.

## 2. Answer questions directly — don't open PRs for non-code topics

If a question is about GitHub plans, billing, platform limits, or general concepts rather than a concrete code change, answer it directly in chat.  Do not open a pull request to "document" the answer.

## 3. Never include secrets in committed content

Do not commit API keys, tokens, passwords, or other credentials to any repository.  If a secret is accidentally exposed, treat rotation as the immediate next step and document the incident.

## 4. Clarify intent before taking write actions

Before performing any action that modifies a repository (push, PR creation, branch deletion, force push, etc.), confirm the user's intent if there is any ambiguity.

## 5. Mobile-first workflows where relevant

When changes touch UI or UX surfaces, prefer patterns and layouts that work well on mobile devices before adapting for larger screens.
