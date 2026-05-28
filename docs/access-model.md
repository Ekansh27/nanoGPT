# Repository Access Model

This document captures who can do what in `Ekansh27/nanoGPT` and why. Written as part of TKK-005.

## Roles & Access Levels

This is a solo learning fork of [karpathy/nanoGPT](https://github.com/karpathy/nanoGPT). The role structure reflects that.

| Role | Who | Permissions | Why |
|---|---|---|---|
| **Admin (owner)** | `@Ekansh27` | Full control: settings, secrets, protection rules, force-push (subject to branch protection), repo deletion | Sole maintainer of the fork. Needs full control to configure and learn from the repo. |
| **Collaborators** | *none currently* | n/a | No teammates on this fork. If added later, default would be **Write** access (push to feature branches, open PRs) — never Admin unless they need to manage settings/secrets. |
| **Upstream** | `@karpathy` (read-only) | None on this fork. We pull *from* his repo via the `upstream` remote, we never push *to* it. | He owns the original; this fork is downstream. |

### How to add a collaborator (for reference)

```bash
gh api -X PUT /repos/Ekansh27/nanoGPT/collaborators/USERNAME -f permission=push
```

Permissions options: `pull` (read), `triage`, `push` (write), `maintain`, `admin`. Default to `push` — collaborators get to work, but only the owner manages governance.

## Branch Protection — `master`

Configured via GitHub branch protection API. Current rules:

- **Require pull request before merging** — direct `git push origin master` is blocked.
- **Required approving reviews:** 0 — practical for a solo repo where there's nobody to review. Realistic teams would set this to 1+ once collaborators exist.
- **Admins bypass:** allowed (`enforce_admins: false`) — without this, a solo owner literally can't merge their own PRs.
- **Force pushes:** blocked.
- **Branch deletion:** blocked.

### Why these settings

- Even with no reviewer, requiring a PR enforces a paper trail: every change to `master` has an associated PR with title, description, diff, and commit history. This is the single most useful governance habit for solo and small-team work.
- Force pushes are disabled because rewriting `master`'s history would invalidate clones and break any branch based on it. Worth the cost only in emergencies, which would be handled by temporarily relaxing the rule.
- Admin bypass is enabled deliberately — required for a solo workflow. If/when a collaborator joins, the bypass should be removed and the review count raised to 1.

## Secrets

Repository secrets are managed via **Settings → Secrets and variables → Actions** (or `gh secret set`).

| Secret | Purpose |
|---|---|
| `EXAMPLE_API_KEY` | Placeholder set during TKK-005 to demonstrate secret management. Not used by any workflow. |

### Rules

- **Never commit secrets to the repo** — that's why this mechanism exists. Secrets are injected into CI/CD at runtime, not stored in code.
- Real secrets (API keys, deploy tokens, model API keys like the Anthropic key) should rotate periodically and have the smallest scope possible.
- Only admins can create/read/delete secrets. Collaborators with `push` access can *use* secrets in workflows but cannot view their values.

## Evolution Plan

This access model is right-sized for solo work. If the repo grows:

1. **First collaborator joins** → add them at `push` permission. Bump `required_approving_review_count` from 0 to 1. Remove admin bypass.
2. **CI/CD added** → start adding required status checks (lint, tests) to branch protection.
3. **More secrets needed** → namespace them clearly (`PROD_*`, `STAGING_*`, `DEV_*`) and document each one's purpose in this file.
4. **Sensitive collaborator** → use `maintain` role rather than `admin`; they can manage issues/PRs but not change governance.

The principle: the rules should be as light as possible while still forcing every change to `master` through a reviewable, reversible path.
