<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-issue/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-issue/v3.0.1** was hardened automatically. 13 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ ... }} expressions are directly interpolated inside run: shell command strings in action.yml. This allows an attacker-controlled value to be parsed by the shell before quoting can protect it.

In the 'Set parameters' step (action.yml):
- Line 27: `if [ -n "${{ inputs.comment }}" ]` — inputs.comment interpolated directly
- Line 28: `comment="--comment \"${{ inputs.comment }}\""` — inputs.comment interpolated directly
- Line 33: `if [ "${{ inputs.close-reason }}" == "not_planned" ]` — inputs.close-reason interpolated directly
- Line 39: `if [ -n "${{ inputs.labels }}" ]` — inputs.labels interpolated directly
- Line 40: `labels=$(echo "${{ inputs.labels }}" | ...)` — inputs.labels interpolated directly

In the 'Close Issue' step (action.yml):
- Line 46: `gh issue close -R "${{ inputs.repository }}"` — inputs.repository interpolated directly
- Line 47: `--reason "${{ steps.params.outputs.close-reason }}"` — steps output interpolated directly
- Line 48: `${{ steps.params.outputs.comment }}` — steps output interpolated directly (also unquoted)
- Line 49: `"${{ inputs.issue-number }}"` — inputs.issue-number interpolated directly

In the 'Add Labels' step (action.yml):
- Line 54: `gh issue edit -R "${{ inputs.repository }}"` — inputs.repository interpolated directly
- Line 55: `--add-label "${{ steps.params.outputs.labels }}"` — steps output interpolated directly
- Line 56: `"${{ inputs.issue-number }}"` — inputs.issue-number interpolated directly

In update-major-version.yml 'Tag new target' and 'Push new tag' steps:
- Line 24: `git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}` — workflow_dispatch inputs interpolated directly and unquoted
- Line 25: `git push origin ${{ github.event.inputs.main_version }} --force` — workflow_dispatch input interpolated directly and unquoted

Locations:

- `action.yml:27`
- `action.yml:28`
- `action.yml:33`
- `action.yml:39`
- `action.yml:40`
- `action.yml:46`
- `action.yml:47`
- `action.yml:48`
- `action.yml:49`
- `action.yml:54`
- `action.yml:55`
- `action.yml:56`
- `.github/workflows/update-major-version.yml:24`
- `.github/workflows/update-major-version.yml:25`

### github-env-injection (severity: high)

The 'Set parameters' step in action.yml writes values derived from untrusted inputs to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

(a) `${{ inputs.comment }}` is interpolated directly into the shell variable `$comment`, which is then written to $GITHUB_OUTPUT via `echo "$comment" >> $GITHUB_OUTPUT` (line ~30). An attacker can inject newlines into inputs.comment to poison subsequent GITHUB_OUTPUT entries.

(b) `${{ inputs.labels }}` is interpolated directly into a shell pipeline and the result is written to $GITHUB_OUTPUT via `echo labels=$labels >> $GITHUB_OUTPUT` (line ~42) without sanitization.

(c) `echo close-reason=... >> $GITHUB_OUTPUT` writes literal strings, but the branch condition itself uses `${{ inputs.close-reason }}` directly (line 33), which is a script-injection vector that also affects the output value.

None of these writes are preceded by `printf '%s' "$VAR" | tr -d '\n\r'`.

Locations:

- `action.yml:27`
- `action.yml:30`
- `action.yml:39`
- `action.yml:42`

### unpinned-uses (severity: high)

All `uses:` references in workflow files use mutable version tags instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised.

Failing references:
- automerge-dependabot.yml: `uses: peter-evans/enable-pull-request-automerge@v3`
- ci.yml: `uses: actions/checkout@v3`
- ci.yml: `uses: actions/upload-artifact@v3`
- ci.yml: `uses: actions/checkout@v3` (test job)
- ci.yml: `uses: actions/download-artifact@v3`
- ci.yml: `uses: peter-evans/create-issue-from-file@v4`
- slash-command-dispatch.yml: `uses: peter-evans/slash-command-dispatch@v3`
- update-major-version.yml: `uses: actions/checkout@v3`

Locations:

- `.github/workflows/automerge-dependabot.yml:10`
- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:21`
- `.github/workflows/ci.yml:31`
- `.github/workflows/ci.yml:32`
- `.github/workflows/ci.yml:38`
- `.github/workflows/slash-command-dispatch.yml:8`
- `.github/workflows/update-major-version.yml:18`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be `write-all` depending on repository settings), granting unnecessary access to the GITHUB_TOKEN.

- automerge-dependabot.yml: No permissions block at top-level or job level. The workflow uses a PAT (ACTIONS_BOT_TOKEN) but the implicit GITHUB_TOKEN permissions are still unrestricted.
- slash-command-dispatch.yml: No permissions block at top-level or job level.
- update-major-version.yml: No permissions block at top-level or job level.

Locations:

- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/update-major-version.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.comment }}" appears directly in run: block of step "Set parameters"; move to env: map

Locations:

- `action.yml:33`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.comment }}" appears directly in run: block of step "Set parameters"; move to env: map

Locations:

- `action.yml:34`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.close-reason }}" appears directly in run: block of step "Set parameters"; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.labels }}" appears directly in run: block of step "Set parameters"; move to env: map

Locations:

- `action.yml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.labels }}" appears directly in run: block of step "Set parameters"; move to env: map

Locations:

- `action.yml:49`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.repository }}" appears directly in run: block of step "Close Issue"; move to env: map

Locations:

- `action.yml:58`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.issue-number }}" appears directly in run: block of step "Close Issue"; move to env: map

Locations:

- `action.yml:61`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.repository }}" appears directly in run: block of step "Add Labels"; move to env: map

Locations:

- `action.yml:69`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.issue-number }}" appears directly in run: block of step "Add Labels"; move to env: map

Locations:

- `action.yml:71`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across action.yml and the four workflow files:

1. action.yml (script-injection, github-env-injection, static-inline-injection): All ${{ inputs.* }} and ${{ steps.*.outputs.* }} expressions moved to env: blocks. Shell scripts use plain $VAR references. Values written to $GITHUB_OUTPUT are sanitized with `printf '%s' | tr -d '\n\r'`. The --comment flag in 'Close Issue' uses a bash array to keep flag and value as separate arguments.

2. automerge-dependabot.yml: Added `permissions: {pull-requests: write, contents: write}` and pinned peter-evans/enable-pull-request-automerge to SHA a660677d5469627102a1c1e11409dd063606628d.

3. ci.yml: Pinned all four action references to full SHA digests (actions/checkout, actions/upload-artifact, actions/download-artifact, peter-evans/create-issue-from-file).

4. slash-command-dispatch.yml: Added `permissions: {issues: read, pull-requests: read}` and pinned peter-evans/slash-command-dispatch to SHA f996d7b7aae9059759ac55e978cff76d91853301.

5. update-major-version.yml: Added `permissions: {contents: write}`, pinned actions/checkout to SHA a37ce9120846195fa4ece8f58b268e6043cb2f26, and moved workflow_dispatch inputs into env: blocks to fix script injection in 'Tag new target' and 'Push new tag' steps.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/ci.yml at line 38. Moved the `${{ matrix.os }}` expression out of the `run:` shell command and into an `env:` block as `OS: ${{ matrix.os }}`. The shell command now uses `$OS` instead of the direct template expression, preventing shell metacharacters in the value from being executed.

