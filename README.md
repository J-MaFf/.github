# .github

Shared GitHub Actions workflows, reusable across my repos.

## claude.yml

The full Claude Code Action job — `@claude` mention filter, permissions, git-policies
fetch into CLAUDE.md, and `anthropics/claude-code-action@v1`. Version bumps (checkout,
the action itself) happen here once instead of once per repo.

### Adopt in another repo

Replace the repo's `.github/workflows/claude.yml` with just the triggers and a stub:

```yaml
name: Claude Code

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]
  pull_request_review:
    types: [submitted]

jobs:
  claude:
    uses: J-MaFf/.github/.github/workflows/claude.yml@main
    permissions:
      contents: write
      pull-requests: write
      issues: write
      id-token: write
      actions: read
    secrets: inherit
```

The `@claude` mention filter lives inside the reusable workflow (the `github` context in
a called workflow is the caller's), so callers don't repeat it. Each repo needs the
`CLAUDE_CODE_OAUTH_TOKEN` secret set (generated via `claude setup-token`). The job never
runs for `homelab-sandbox-agent[bot]`, the claude-sandbox App, so the App cannot reach the
token through an `@claude` mention:

```bash
gh secret set CLAUDE_CODE_OAUTH_TOKEN --repo <owner>/<repo>
```

The explicit `permissions` block matters: it guarantees the called job gets write access
even in repos whose default workflow token is read-only.

## ps-lint.yml

PowerShell static lint on the self-hosted Windows runner: parse-check every
`*.ps1`/`*.psm1`, then PSScriptAnalyzer failing on **Error** severity only (warnings are
reported, not fatal). Static analysis only — it never executes repo scripts.

### Adopt in another repo

```yaml
name: PowerShell Lint

on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint:
    uses: J-MaFf/.github/.github/workflows/ps-lint.yml@main
```

Optionally scope which directories are linted (space-separated, missing ones skipped):

```yaml
    with:
      paths: scripts biometric-sudo-ssh
```

No secrets required. The caller repo must have the self-hosted `windows` runner
registered. Repo-specific checks (module-manifest validation, smoke tests, Pester
suites) stay in the caller as additional jobs.

## merge-gate.yml

The required check for repos enrolled in the claude-sandbox overnight runs
(J-MaFf/homelab `claude-sandbox/`). A PR the sandbox GitHub App
(`homelab-sandbox-agent[bot]`) authored, or authored or committed any commit of, passes
only once the approver (`J-MaFf`) has approved its current head. PRs the App never
touched pass at once. On a review, a second job re-runs the pull_request-event run on the
head SHA when its verdict is stale, since that run is the one the ruleset sees.

### Adopt in another repo

The file **must** be `.github/workflows/merge-gate.yml` (the re-run job looks up the
caller's runs by that file name) and the job **must** be `merge-gate`, so the check
reports as `merge-gate / merge-gate`:

```yaml
name: merge-gate

on:
  pull_request:
    types: [opened, synchronize, reopened]
  pull_request_review:
    types: [submitted, dismissed]

permissions: {}

jobs:
  merge-gate:
    uses: J-MaFf/.github/.github/workflows/merge-gate.yml@main
    permissions:
      contents: read
      pull-requests: read
      actions: write
```

`app` and `approver` inputs override the defaults. Then require the check in the repo's
`main` ruleset as context `merge-gate / merge-gate`, integration_id `15368` (GitHub
Actions). In the homelab setup `claude-sandbox/github/enroll-repo.sh` does both.

## discord-notify.yml

Posts a CI failure message to Discord — repository, workflow name, branch/ref, and a
link to the failed run. Only fires when the caller job actually failed; green runs post
nothing.

### Adopt in another repo

Add a final `notify` job to your workflow, listing every job it should watch:

```yaml
  notify:
    needs: [<your other job names>]
    if: failure()
    uses: J-MaFf/.github/.github/workflows/discord-notify.yml@main
    secrets: inherit
```

Set the webhook once per repo (the reusable workflow reads it as `DISCORD_CI_WEBHOOK`):

```bash
gh secret set DISCORD_CI_WEBHOOK --repo <owner>/<repo>
```

If a repo has more than one independently-triggered workflow (e.g. split by path
filters), give each one its own `notify` job rather than trying to share one across
workflows — `needs` can only reference jobs in the same workflow file.

### Behavior

- Requires the `DISCORD_CI_WEBHOOK` secret; the notify job fails visibly if it's unset
  or the POST doesn't succeed — a broken notifier won't look green.
- Posts plain-content messages only (no embeds, no mentions, no success/recovery pings).
