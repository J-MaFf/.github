# STATUS — .github

## What This Is
Shared, reusable (`workflow_call`) GitHub Actions workflows for all J-MaFf repos. Version
bumps and behavior changes happen here once instead of once per repo. This repo is
load-bearing: ~25 repos call these workflows from `main`, so changes take effect
fleet-wide on merge.

## Current State — 2026-08-10
Healthy. Three reusable workflows, all adopted fleet-wide.

### Components
| File | Description |
|---|---|
| `.github/workflows/claude.yml` | Full Claude Code Action job (@claude mention filter, permissions, git-policies fetch, `claude-code-action@v1`). Callers keep triggers + a `uses:` stub with `secrets: inherit`. Adopted by ~25 repos. |
| `.github/workflows/ps-lint.yml` | PowerShell static lint: parse-check + PSScriptAnalyzer (Error-severity fail only) on the self-hosted `windows` runner. `paths` input scopes directories. Static-only — never executes repo scripts. |
| `.github/workflows/discord-notify.yml` | Posts CI-failure messages to Discord `#ci`. Callers add a `notify` job with `if: failure()`. |
| `README.md` | Adoption instructions for each workflow. |

### Resolved Issues
| Issue | Description | PR |
|---|---|---|
| [#1](https://github.com/J-MaFf/.github/issues/1) | Discord notify showed PR merge ref instead of branch | [#2](https://github.com/J-MaFf/.github/pull/2) |
| [#3](https://github.com/J-MaFf/.github/issues/3) | Add reusable claude.yml and ps-lint.yml workflows | [#4](https://github.com/J-MaFf/.github/pull/4) |

### Open Issues
None.

## Natural Next Steps
- When bumping `actions/checkout` or `claude-code-action` versions, bump here — the fleet
  inherits it.
- If a caller repo's ruleset requires a status check by name, remember reusable-workflow
  checks report as `<caller job> / <called job>` (e.g. `validate / ps-lint`).

## Prerequisites to Run
Nothing runs here directly. Callers need: `CLAUDE_CODE_OAUTH_TOKEN` secret (claude.yml),
`DISCORD_CI_WEBHOOK` secret (discord-notify.yml), and a registered self-hosted `windows`
runner (ps-lint.yml). See `README.md` for adoption snippets.
