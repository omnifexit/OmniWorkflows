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
- Dependabot merges re-run CI on `main` (see `dependabot-auto-merge.yml`
  below), so they are promoted the same way.

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

## dependabot-auto-merge.yml

Runs at the end of each repo's CI on Dependabot pull requests, once every
check on the pull request has passed:

- **Minor and patch** updates are squash-merged, then CI is started on `main`
  (merges made with `GITHUB_TOKEN` don't trigger workflows), so they are tested
  and promoted to `production` like any other push.
- **Everything else** (majors, also of development dependencies, since those
  can need code changes too) stays open: assigned to `anass1090`, so GitHub
  notifies them, with a comment listing what changes. Nobody is notified about
  updates that merge on their own.

### Adding it to a repo

Make sure the CI workflow has `workflow_dispatch:` in its triggers (also needed
for promotion), then add this job at the end of it:

```yaml
  dependabot:
    needs: [<all test jobs>]
    if: github.event_name == 'pull_request' && github.event.pull_request.user.login == 'dependabot[bot]'
    permissions:
      actions: write
      contents: write
      pull-requests: write
    uses: omnifexit/OmniWorkflows/.github/workflows/dependabot-auto-merge.yml@main
    with:
      ci-workflow: ci.yml   # this workflow's file name
```

Optional input: `reviewer` (default `anass1090`).

## Access

This repo is public, so the other repos' workflows and Dependabot can read it
without any access setting. It holds no secrets: never add any here.
