# OmniWorkflows

Shared GitHub Actions workflows for the OmnifexIT repos. Only used by CI; you
never need this repo to run or develop a project locally.

## promote-to-production.yml

Every repo deploys through Coolify, which watches the repo's `production`
branch (not `main`). Each repo's CI calls this workflow after all tests,
formatting checks and builds pass on `main`, and it fast-forwards `production`
to that commit, which makes Coolify deploy it.

```
push to main → CI (tests, Pint, Prettier, build) → green → production moves → Coolify deploys
```

- Never commit to `production` directly; push to `main`.
- If CI fails, `production` stays on the last green commit.
- Dependabot merges re-run CI on `main` (see the `dependabot-auto-merge` job
  in each repo), so they are promoted the same way.

### Adding it to a new repo

1. Create `production` at the current `main` and point the Coolify resource(s)
   at the `production` branch.
2. Add `workflow_dispatch:` to the CI triggers and this job at the end of `ci.yml`:

```yaml
  promote:
    needs: [<all test jobs>]
    if: github.ref == 'refs/heads/main' && github.event_name != 'pull_request'
    permissions:
      contents: write
    uses: omnifexit/OmniWorkflows/.github/workflows/promote-to-production.yml@main
```

This repo is private; it is shared with the other repos through
Settings → Actions → General → Access ("Accessible from repositories owned by
the user").
