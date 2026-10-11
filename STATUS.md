# STATUS — .github

## What This Is
Shared, reusable (`workflow_call`) GitHub Actions workflows for all J-MaFf repos. Version
bumps and behavior changes happen here once instead of once per repo. This repo is
load-bearing: ~25 repos call these workflows from `main`, so changes take effect
fleet-wide on merge.

## Current State — 2026-10-11
Healthy. Four fleet-wide reusable workflows, plus `merge-gate.yml` for the repos enrolled
in the claude-sandbox overnight runs (J-MaFf/homelab `claude-sandbox/`).

### Components
| File | Description |
|---|---|
| `.github/workflows/claude.yml` | Full Claude Code Action job (@claude mention filter, permissions, git-policies fetch, `claude-code-action@v1`). Never runs for `homelab-sandbox-agent[bot]`, the claude-sandbox App. Callers keep triggers + a `uses:` stub with `secrets: inherit`. Adopted by ~25 repos. |
| `.github/workflows/ps-lint.yml` | PowerShell static lint: parse-check + PSScriptAnalyzer (Error-severity fail only) on the self-hosted `windows` runner. `paths` input scopes directories. Static-only — never executes repo scripts. |
| `.github/workflows/discord-notify.yml` | Posts CI-failure messages to Discord `#ci`. Callers add a `notify` job with `if: failure()`. |
| `.github/workflows/merge-gate.yml` | Required check for repos the claude-sandbox App (`homelab-sandbox-agent[bot]`) works in: a PR it authored or committed to passes only once the approver approved the current head; a review re-runs the stale pull_request-event run. Callers name the file `merge-gate.yml` and the job `merge-gate`; rulesets require `merge-gate / merge-gate` (integration 15368). |
| `.github/workflows/release.yml` | Builds a filtered release zip from caller-specified `include-paths` and publishes (or, if the tag was re-pushed, updates) a GitHub Release for the tag that triggered the calling workflow. Callers keep a `push: tags:` trigger + a `uses:` stub. |
| `README.md` | Adoption instructions for each workflow. |

### Resolved Issues
| Issue | Description | PR |
|---|---|---|
| [#1](https://github.com/J-MaFf/.github/issues/1) | Discord notify showed PR merge ref instead of branch | [#2](https://github.com/J-MaFf/.github/pull/2) |
| [#3](https://github.com/J-MaFf/.github/issues/3) | Add reusable claude.yml and ps-lint.yml workflows | [#4](https://github.com/J-MaFf/.github/pull/4) |
| [#13](https://github.com/J-MaFf/.github/issues/13) | release.yml failed whenever a re-pushed tag already had a release | [#14](https://github.com/J-MaFf/.github/pull/14) |

### Open Issues
| Issue | Description |
|---|---|
| [#15](https://github.com/J-MaFf/.github/issues/15) | Reusable merge-gate.yml and the claude.yml sandbox-App actor filter (in review) |

## Natural Next Steps
- When bumping `actions/checkout` or `claude-code-action` versions, bump here — the fleet
  inherits it.
- If a caller repo's ruleset requires a status check by name, remember reusable-workflow
  checks report as `<caller job> / <called job>` (e.g. `validate / ps-lint`).

## Prerequisites to Run
Nothing runs here directly. Callers need: `CLAUDE_CODE_OAUTH_TOKEN` secret (claude.yml),
`DISCORD_CI_WEBHOOK` secret (discord-notify.yml), and a registered self-hosted `windows`
runner (ps-lint.yml). merge-gate.yml needs no secrets; a caller must pass
`actions: write` so the review run can re-run the pull_request-event run. See `README.md` for adoption snippets.
