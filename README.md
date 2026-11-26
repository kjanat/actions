# GitHub triage

`kjanat/actions@triage` classifies issues and pull requests with TypeSafe. It uses the target repository's label catalog and, for issues, can also suggest an issue type, priority, effort, start date, and target date when those fields are available. It prints the proposed changes as JSON and applies them by default. Set `apply: "false"` to preview them.

The action sends excerpts of the title, body, and pull request file paths to TypeSafe. It runs its bundled script and needs no checkout step in the caller workflow.

## Caller workflow

Save this as `.github/workflows/triage.yml` in the repository to triage.

```yaml
name: Triage issues and pull requests
on:
  issues:
    types: [opened, edited, reopened]
  pull_request_target:
    types: [opened, edited, reopened, synchronize]
  workflow_dispatch:
    inputs:
      number:
        description: Issue or pull request number
        required: true
        type: string

permissions:
  issues: write
  pull-requests: write

concurrency:
  group: triage-${{ github.repository }}-${{ github.event.issue.number || github.event.pull_request.number || inputs.number }}
  cancel-in-progress: false

jobs:
  triage:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: kjanat/actions@triage
        with:
          typesafe-api-key: ${{ secrets.TYPESAFE_API_KEY }}
          labels: ${{ vars.TRIAGE_LABELS }}
          target: ${{ inputs.number && format('{0}#{1}', github.repository, inputs.number) || '' }}
```

| Input | Default | Purpose |
| --- | --- | --- |
| `typesafe-api-key` | `${{ env.TYPESAFE_API_KEY }}` | Credential for the TypeSafe classifier. A nonempty input takes precedence. |
| `token` | `${{ env.GITHUB_TOKEN \|\| env.GH_TOKEN \|\| github.token }}` | A nonempty input takes precedence. Override when the target or its issue metadata requires a different token. |
| `target` | Issue or PR that triggered the workflow | Accepts `owner/repo#123` or a GitHub issue or PR URL. Supply it explicitly for manual or scheduled runs. |
| `labels` | All existing labels are candidates | Provide a JSON array such as `["bug","documentation"]` to limit classification to topical labels. The `other` fallback is handled separately. |
| `apply` | `"true"` | Set to `"false"` to preview. Preview still calls TypeSafe. |

## TypeSafe credential

Set `TYPESAFE_API_KEY` as a secret in the caller repository. Pass it through `typesafe-api-key` as above, or set `TYPESAFE_API_KEY: ${{ secrets.TYPESAFE_API_KEY }}` in the caller's `env`. If Beepboop already holds the key, use its repository-secret delivery for that repository. The Beepboop runtime credentials action requires an active release execution. See [Beepboop credential delivery](https://github.com/kjanat/beepboop/blob/master/docs/automation.md#credentials).
