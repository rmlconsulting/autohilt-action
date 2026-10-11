# autoHILT run — hardware tests from GitHub Actions

One step in your test repository's workflow: the build says what to test,
your autoHILT fleet runs it on production stations, and the step ends with
the verdict.

```yaml
name: hardware tests
on: [pull_request]
jobs:
  hil:
    runs-on: ubuntu-latest
    steps:
      - uses: rmlconsulting/autohilt-action@v1
        with:
          run: smoke on 2 units
          token: ${{ secrets.AUTOHILT_TOKEN }}
```

- **`run`** is the request as you would say it to the Slack bot: `smoke on
  2 units`, `product:widget on all`, `-k uart on station-004`, `nightly 3x`.
  The build's commit is pinned (`ref`, the build's sha by default); the
  machines fetch it, run, and restore their checkout.
- **`token`** is the company's CI token: a test admin mints it on
  **Admin → CI access** and stores it as the repository secret
  `AUTOHILT_TOKEN`. It belongs to the company (it outlives any one
  person's membership), reaches only the run doors, and never lands on
  anyone's dev machine.
- **Waiting is the design**: a reserved or busy station queues the run
  behind the current party; the step prints what each run waits on.
  `timeout` (default 60 minutes) bounds the wait, and at the bound
  `cancel_on_timeout` (default `true`) cancels what has not started and
  reports what finished — a running test always finishes on the machine.
- **The verdict** fails the job on a failed test, writes the job summary
  (station · status · duration · failed tests) and sets the outputs
  `batch_id`, `verdict`, `passed`, `failed`, `url`.

Under the step runs the engine's own `autohilt run` (`pip install
autohilt`), so the same thing works from a terminal and from any CI:

```bash
AUTOHILT_CI_TOKEN=ahw_… autohilt run "smoke on 2 units" --ref "$GITHUB_SHA"
```

MIT licensed. The source of this action is `actions/autohilt-run` in the autoHILT repository; this repository is published from it by release and never edited directly.
