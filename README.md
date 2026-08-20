# workflows

Reusable GitHub Actions workflows shared across my repositories.

## anti-slop — AI slop PR protection

`.github/workflows/anti-slop.yml` wraps [peakoss/anti-slop](https://github.com/peakoss/anti-slop),
a GitHub Action that runs ~34 quality checks on incoming pull requests
(branch, size, title, description, commits, files, user profile, merge
history) and closes PRs that look like low-quality or AI-generated slop.

### Why this repo exists

Instead of every repository pinning and configuring the action separately,
they call one reusable workflow. The action is pinned to a commit SHA here
— bumping the pin in this repo updates every caller at once.

### Inputs

| Input | Type | Default | Notes |
|-|-|-|-|
| `max-failures` | number | `4` | Check failures needed before the PR is acted on. The action's own default. Valid range 1–25. |
| `close-pr` | boolean | `true` | `false` gives you a dry run: all checks still report into the job summary, nothing gets closed. Worth doing first on any repo with real traffic. |
| `failure-pr-message` | string | `""` | Comment posted when a PR is closed. Empty means a closed contributor gets no explanation — worth setting. |

### Calling it from another repository

Minimal caller — defaults everywhere:

```yaml
name: PR Quality

on:
  pull_request_target:
    types: [opened, reopened]

permissions:
  contents: read
  issues: read
  pull-requests: write

jobs:
  pr-quality:
    uses: ksaltnes/workflows/.github/workflows/anti-slop.yml@main
```

Caller with a dry-run first pass and an explanation message:

```yaml
name: PR Quality

on:
  pull_request_target:
    types: [opened, reopened]

permissions:
  contents: read
  issues: read
  pull-requests: write

jobs:
  pr-quality:
    uses: ksaltnes/workflows/.github/workflows/anti-slop.yml@main
    with:
      max-failures: 6
      close-pr: false
      failure-pr-message: >-
        This PR was closed automatically by a quality gate. See the job
        summary for which checks failed. If this is a mistake, open an
        issue and we will take a look.
```

Notes:

- The caller must grant `contents: read`, `issues: read`,
  `pull-requests: write`. A called workflow can only narrow the
  permissions its caller granted, never widen them — the reusable
  workflow declares its own permissions and fails loudly if they are
  missing.
- `pull_request_target` runs in the context of the base repository, so
  the workflow can close PRs from forks that have no write access.
- Consider pinning the `uses:` line to a commit SHA of this repo instead
  of `@main` once you want caller-side stability.
