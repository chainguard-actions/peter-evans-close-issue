<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-issue/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-issue/v3.0.1** was hardened automatically. 13 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 'Set parameters' (action.yml): Multiple ${{ }} expressions from `inputs.*` are interpolated directly into the run: shell script without going through env: variables. This allows an attacker-controlled input to inject arbitrary shell commands. Offending lines:
- Line 31: `if [ -n "${{ inputs.comment }}" ]; then`
- Line 32: `comment="--comment \"${{ inputs.comment }}\""`
- Line 39: `if [ "${{ inputs.close-reason }}" == "not_planned" ]; then`
- Line 45: `if [ -n "${{ inputs.labels }}" ]; then`
- Line 46: `labels=$(echo "${{ inputs.labels }}" | tr '\n' ',' | sed 's/,$//')`
Violates sub-rule (a): direct expression interpolation inside run: block.

Locations:

- `action.yml:31`
- `action.yml:32`
- `action.yml:39`
- `action.yml:45`
- `action.yml:46`

### script-injection (severity: high)

Step 'Close Issue' (action.yml): Multiple ${{ }} expressions from `inputs.*` and `steps.*.outputs.*` are interpolated directly into the run: shell script. Notably, `${{ steps.params.outputs.comment }}` is interpolated completely unquoted, making shell metacharacter injection trivial. Offending lines:
- Line 55: `gh issue close -R "${{ inputs.repository }}"`
- Line 56: `--reason "${{ steps.params.outputs.close-reason }}"`
- Line 57: `${{ steps.params.outputs.comment }}` (unquoted — sub-rule (a) and (b))
- Line 58: `"${{ inputs.issue-number }}"`
Violates sub-rule (a): direct expression interpolation inside run: block.

Locations:

- `action.yml:55`
- `action.yml:56`
- `action.yml:57`
- `action.yml:58`

### script-injection (severity: high)

Step 'Add Labels' (action.yml): Multiple ${{ }} expressions from `inputs.*` and `steps.*.outputs.*` are interpolated directly into the run: shell script. Offending lines:
- Line 64: `gh issue edit -R "${{ inputs.repository }}"`
- Line 65: `--add-label "${{ steps.params.outputs.labels }}"`
- Line 66: `"${{ inputs.issue-number }}"`
Violates sub-rule (a): direct expression interpolation inside run: block.

Locations:

- `action.yml:64`
- `action.yml:65`
- `action.yml:66`

### github-env-injection (severity: high)

Step 'Set parameters' (action.yml): Values derived from untrusted inputs are written to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically:
- `inputs.comment` is stored in the shell variable `$comment` (line 32) and then written to $GITHUB_OUTPUT via a heredoc-style delimiter block (lines 34–36). An attacker can inject newlines into `inputs.comment` to poison the output file.
- `inputs.labels` is processed and stored in `$labels` (line 46) and written to $GITHUB_OUTPUT (line 49) without sanitization.
This allows an attacker to inject arbitrary key=value pairs into $GITHUB_OUTPUT, potentially overwriting other outputs or injecting environment variables.

Locations:

- `action.yml:34`
- `action.yml:35`
- `action.yml:49`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote action.yml to fix all script injection and github-env-injection findings:

1. Set parameters step: Moved inputs.comment, inputs.close-reason, and inputs.labels to env: block as INPUT_COMMENT, INPUT_CLOSE_REASON, INPUT_LABELS. Shell script uses only $INPUT_* variables. Labels are sanitized with 'printf | tr -d \n\r' before writing to GITHUB_OUTPUT. Comment is stored as raw text via heredoc (random delimiter prevents injection; per rules, no newline stripping for heredoc-delimited values).

2. Close Issue step: Moved inputs.repository, inputs.issue-number, steps.params.outputs.close-reason, and steps.params.outputs.comment to env: block. The previously unquoted ${{ steps.params.outputs.comment }} is now handled via a bash array (args+=(--comment "$PARAMS_COMMENT")) to keep flag and value as separate arguments.

3. Add Labels step: Moved inputs.repository, inputs.issue-number, and steps.params.outputs.labels to env: block. Shell script uses only $INPUT_* and $PARAMS_* variables.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set parameters' step in action.yml: replaced the heredoc delimiter approach for writing the `comment` output with a sanitized single-line write. The comment value is now sanitized using `printf '%s' "$INPUT_COMMENT" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT as `echo "comment=$safe_comment" >> $GITHUB_OUTPUT`. This prevents newline injection attacks where an attacker could smuggle additional key=value pairs into the GitHub output context. The fix follows the same pattern already used for the `labels` output in the same step.

