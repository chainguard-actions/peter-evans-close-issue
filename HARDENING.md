<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-issue/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-issue/v3.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In update-major-version.yml, attacker-controlled workflow_dispatch inputs are interpolated directly: `git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}` and `git push origin ${{ github.event.inputs.main_version }} --force`. These values flow through YAML template substitution before the shell sees them, enabling command injection. In ci.yml, `${{ matrix.os }}` is interpolated directly into `run: echo "#[CI] test ${{ matrix.os }}" > issue.md`.

Locations:

- `.github/workflows/update-major-version.yml:27`
- `.github/workflows/update-major-version.yml:29`
- `.github/workflows/ci.yml:30`

### github-env-injection (severity: high)

In action.yml's 'Set parameters' step, inputs.labels is mapped to the env var LABELS (${{ inputs.labels }}), then written to $GITHUB_OUTPUT via `echo labels="$labels" >> $GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline in the labels input could inject arbitrary key=value pairs into GITHUB_OUTPUT. Similarly, inputs.close-reason is mapped to CLOSE_REASON and its derived value is written to $GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:38`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable version tags instead of pinned 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: automerge-dependabot.yml: `peter-evans/enable-pull-request-automerge@v3`; ci.yml: `actions/checkout@v6`, `actions/upload-artifact@v7`, `actions/checkout@v6` (again), `actions/download-artifact@v8`, `peter-evans/create-issue-from-file@v6`; slash-command-dispatch.yml: `peter-evans/slash-command-dispatch@v5`; update-major-version.yml: `actions/checkout@v6`.

Locations:

- `.github/workflows/automerge-dependabot.yml:9`
- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:28`
- `.github/workflows/ci.yml:29`
- `.github/workflows/ci.yml:36`
- `.github/workflows/slash-command-dispatch.yml:8`
- `.github/workflows/update-major-version.yml:19`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions: automerge-dependabot.yml, slash-command-dispatch.yml, and update-major-version.yml.

Locations:

- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/update-major-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings:
1. script-injection: Moved ${{ github.event.inputs.* }} expressions in update-major-version.yml and ${{ matrix.os }} in ci.yml into env: blocks, referencing them as plain shell variables.
2. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing close-reason and labels to $GITHUB_OUTPUT in action.yml.
3. unpinned-uses: Pinned all 6 unpinned action references to full 40-char SHA digests with tag comments (peter-evans/enable-pull-request-automerge@v3, actions/checkout@v6 x2, actions/upload-artifact@v7, actions/download-artifact@v8, peter-evans/create-issue-from-file@v6, peter-evans/slash-command-dispatch@v5).
4. missing-permissions: Added minimal permissions blocks to automerge-dependabot.yml (pull-requests: write, contents: write), slash-command-dispatch.yml (issues: read, pull-requests: read), and update-major-version.yml (contents: write).

