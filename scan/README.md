# autoHILT scan — which rig parts each test needs, from GitHub Actions

`autohilt init` writes this workflow into your test repository
(`.github/workflows/autohilt-scan.yml`). You set up nothing else:

```yaml
name: autoHILT scan
on:
  push:
    branches: ["**"]
  workflow_dispatch:
    inputs:
      os:
        type: choice
        options: [linux, windows, macos]
        default: linux
permissions:
  contents: read
  id-token: write
jobs:
  scan:
    runs-on: ${{ inputs.os == 'windows' && 'windows-latest' || inputs.os == 'macos' && 'macos-latest' || 'ubuntu-latest' }}
    steps:
      - uses: rmlconsulting/autohilt-action/scan@v1
```

- **Every push scans its commit on Linux.** autoHILT starts the same
  workflow on Windows or macOS when a run needs to know a test's parts
  there (a test that only loads on Windows, say).
- **It runs none of your tests and touches no hardware.** It collects your
  tests the way pytest does and reports test names, markers and the names
  of the rig parts each test uses, so autoHILT runs a test only on a unit
  that has them. None of your code leaves GitHub.
- **No secret.** GitHub's own token for the job (`id-token: write`) proves
  to autoHILT which repository and commit the report is for. autoHILT takes
  it only for a repository your company linked as a test repository, with
  the autoHILT GitHub App installed on its owner.
- **Fast to start.** The commit alone (no older history), uv for Python and
  the engine, and none of your repository's own dependencies: an import the
  runner cannot satisfy is stood in for during the scan.

Inputs: `path` (the folder holding `autohilt.toml`, default the top),
`server` (default: the one your `autohilt.toml` names), `python` (default
3.12), `engine` (default `autohilt>=0.16.0`).
