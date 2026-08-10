# .github

Shared GitHub Actions workflows, reusable across my repos.

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
