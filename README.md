# retry-failed-jobs

A GitHub Action to rerun the failed jobs of a finished workflow run when every failure is marked as retryable, such as a transient network error or a busy shared resource.

A step marks a failure as retryable by writing an annotation (error, warning or notice) whose title is one of the `markers`:

```bash
echo "::error title=transient-failure::Download timed out"
```

The failed jobs are rerun only when every failed job, other than `ignore-jobs`, carries a marker. A rerun repeats every failed job, so a single real failure leaves the run failed. The rerun is skipped if the run has moved on to another attempt since it was checked. Cancelled jobs are not failures, so a cancelled run reruns nothing, while jobs cancelled by `fail-fast` alongside a marked failure are rerun with it.

## How to use

### workflow

Run it from a `workflow_run` workflow, which starts only once the watched run has finished, so nothing waits or polls in between.

```yaml
on:
  workflow_run:
    workflows: [<workflow name>]
    types: [completed]
jobs:
  rerun:
    if: github.event.workflow_run.conclusion == 'failure'
    runs-on: ubuntu-latest
    permissions:
      actions: write # to rerun the failed jobs
      checks: read # to read the failed jobs' annotations
    steps:
      - uses: RageAgainstThePixel/retry-failed-jobs@v1
        with:
          markers: transient-failure
```

For example, [unity-cli](https://github.com/RageAgainstThePixel/unity-cli) marks its retryable failures with `unity-license-exhausted` and `unity-transient-failure`.

### inputs

| name | description | required |
| ---- | ----------- | -------- |
| `markers` | Comma-separated annotation titles that mark a failure as retryable. | true |
| `run-id` | The finished run to check. | Defaults to the run of a `workflow_run` event. |
| `run-attempt` | The attempt of that run that finished. | Defaults to the attempt of a `workflow_run` event. |
| `max-attempts` | The last attempt that may be rerun; later attempts are left failed. | Defaults to `3`. |
| `ignore-jobs` | Comma-separated names of jobs that only report other jobs' results, such as an aggregate check. | Defaults to none. |
| `delay-seconds` | Seconds to wait before the rerun, such as for a shared resource to be released. | Defaults to `0`. |
| `github-token` | Token with `actions: write` and `checks: read` on the repository. | Defaults to `${{ github.token }}`. |

### outputs

| name | description |
| ---- | ----------- |
| `rerun` | `true` when the failed jobs were rerun. |

## Development

Requires Node.js 24.

```bash
npm run build      # clean install + bundle to dist/
npm run typecheck  # tsc --noEmit
```

Commit the rebuilt `dist/` with your changes.
