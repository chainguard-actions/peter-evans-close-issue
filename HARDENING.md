<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-issue/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **peter-evans--close-issue/v3.0.1** was hardened automatically. 19 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ ... }} expressions are directly interpolated inside run: shell command strings in action.yml. In the 'Set parameters' step, ${{ inputs.comment }}, ${{ inputs.close-reason }}, and ${{ inputs.labels }} are interpolated directly into shell code. In the 'Close Issue' step, ${{ inputs.repository }}, ${{ steps.params.outputs.close-reason }}, ${{ steps.params.outputs.comment }}, and ${{ inputs.issue-number }} are interpolated directly. In the 'Add Labels' step, ${{ inputs.repository }}, ${{ steps.params.outputs.labels }}, and ${{ inputs.issue-number }} are interpolated directly. All of these allow an attacker-controlled value to inject arbitrary shell commands.

Locations:

- `action.yml:28`
- `action.yml:45`
- `action.yml:53`

### script-injection (severity: high)

Sub-rule (a): In update-major-version.yml, the workflow_dispatch inputs ${{ github.event.inputs.main_version }} and ${{ github.event.inputs.target }} are directly interpolated into run: shell commands ('git tag -f ...' and 'git push origin ...'). An attacker with write access who can trigger workflow_dispatch can inject arbitrary shell commands via these inputs.

Locations:

- `.github/workflows/update-major-version.yml:24`
- `.github/workflows/update-major-version.yml:26`

### github-env-injection (severity: high)

In action.yml's 'Set parameters' step, untrusted inputs are written to $GITHUB_OUTPUT without sanitization. Specifically: (1) ${{ inputs.comment }} is written via a heredoc delimiter pattern to $GITHUB_OUTPUT; (2) ${{ inputs.close-reason }} is compared and its result echoed to $GITHUB_OUTPUT; (3) ${{ inputs.labels }} is processed and echoed to $GITHUB_OUTPUT. None of these writes are preceded by the required sanitization step (printf '%s' ... | tr -d '\n\r'). A newline-injection attack via any of these inputs could allow an attacker to set arbitrary environment variables or outputs.

Locations:

- `action.yml:28`

### missing-permissions (severity: medium)

automerge-dependabot.yml has no top-level permissions: key and no job-level permissions: key on the 'automerge' job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/automerge-dependabot.yml:1`

### missing-permissions (severity: medium)

slash-command-dispatch.yml has no top-level permissions: key and no job-level permissions: key on the 'slashCommandDispatch' job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/slash-command-dispatch.yml:1`

### missing-permissions (severity: medium)

update-major-version.yml has no top-level permissions: key and no job-level permissions: key on the 'tag' job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/update-major-version.yml:1`

### unpinned-uses (severity: high)

automerge-dependabot.yml references 'peter-evans/enable-pull-request-automerge@v3' — a mutable tag ref, not a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/workflows/automerge-dependabot.yml:9`

### unpinned-uses (severity: high)

ci.yml references multiple actions with mutable tag refs instead of pinned SHA digests: 'actions/checkout@v3' (lines 20, 30), 'actions/upload-artifact@v3' (line 21), 'actions/download-artifact@v3' (line 32), and 'peter-evans/create-issue-from-file@v4' (line 38). These are all vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:21`
- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:32`
- `.github/workflows/ci.yml:38`

### unpinned-uses (severity: high)

slash-command-dispatch.yml references 'peter-evans/slash-command-dispatch@v3' — a mutable tag ref, not a full 40-character commit SHA.

Locations:

- `.github/workflows/slash-command-dispatch.yml:9`

### unpinned-uses (severity: high)

update-major-version.yml references 'actions/checkout@v3' — a mutable tag ref, not a full 40-character commit SHA.

Locations:

- `.github/workflows/update-major-version.yml:19`

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

**Fixes applied:** script-injection, github-env-injection, missing-permissions, unpinned-uses, static-inline-injection

**Notes:**

Fixed all findings across action.yml and workflow files:

1. action.yml (script-injection, github-env-injection, static-inline-injection): Moved all ${{ inputs.* }} and ${{ steps.params.outputs.* }} expressions from run: blocks into env: blocks. Sanitized all GITHUB_OUTPUT writes with printf '%s' ... | tr -d '\n\r'. Used bash array pattern for optional --comment flag to avoid flag+value-in-one-variable anti-pattern.

2. update-major-version.yml (script-injection, missing-permissions, unpinned-uses): Moved ${{ github.event.inputs.* }} into env: blocks; added 'permissions: contents: write'; pinned actions/checkout@v3 to full SHA.

3. automerge-dependabot.yml (missing-permissions, unpinned-uses): Added 'permissions: pull-requests: write, contents: write'; pinned peter-evans/enable-pull-request-automerge@v3 to full SHA.

4. slash-command-dispatch.yml (missing-permissions, unpinned-uses): Added 'permissions: issues: read, pull-requests: read'; pinned peter-evans/slash-command-dispatch@v3 to full SHA.

5. ci.yml (unpinned-uses): Pinned all four action references (actions/checkout@v3, actions/upload-artifact@v3, actions/download-artifact@v3, peter-evans/create-issue-from-file@v4) to full commit SHAs.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/ci.yml line 38: moved `${{ matrix.os }}` out of the `run:` shell command into an `env:` block as `OS: ${{ matrix.os }}`, then updated the shell command to use `$OS` instead of the direct template expression. This prevents the matrix value from being interpolated directly into the shell command string.

