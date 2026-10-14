<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-issue/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-issue/v3.0.1** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ inputs.* }}` and `${{ steps.*.outputs.* }}` expressions inside shell command strings (sub-rule a). An attacker who controls these inputs (e.g. via a calling workflow) can inject arbitrary shell commands.

Affected lines in the "Set parameters" step:
- Line 31: `if [ -n "${{ inputs.comment }}" ]`
- Line 32: `comment="--comment \"${{ inputs.comment }}\""`
- Line 37: `if [ "${{ inputs.close-reason }}" == "not_planned" ]`
- Line 43: `if [ -n "${{ inputs.labels }}" ]`
- Line 44: `labels=$(echo "${{ inputs.labels }}" | tr '\n' ',' | sed 's/,$//')`

Affected lines in the "Close Issue" step:
- Line 52: `gh issue close -R "${{ inputs.repository }}"`
- Line 53: `--reason "${{ steps.params.outputs.close-reason }}"`
- Line 54: `${{ steps.params.outputs.comment }}` (also unquoted — sub-rule b)
- Line 55: `"${{ inputs.issue-number }}"`

Affected lines in the "Add Labels" step:
- Line 63: `gh issue edit -R "${{ inputs.repository }}"`
- Line 64: `--add-label "${{ steps.params.outputs.labels }}"`
- Line 65: `"${{ inputs.issue-number }}"`

Fix: Move all inputs into `env:` variables and reference them as `"$VAR"` in the shell script, never using `${{ }}` directly inside `run:` blocks.

Locations:

- `action.yml:31`
- `action.yml:52`
- `action.yml:63`

### github-env-injection (severity: high)

The "Set parameters" step writes values derived from `inputs.comment` and `inputs.labels` to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

- Line 34: `echo "$comment" >> $GITHUB_OUTPUT` — `$comment` contains `${{ inputs.comment }}` interpolated at YAML-parse time; a newline in the input can inject additional key=value pairs into GITHUB_OUTPUT.
- Line 47: `echo labels=$labels >> $GITHUB_OUTPUT` — `$labels` is derived from `${{ inputs.labels }}`; despite the `tr '\n' ','` transformation, the value is never sanitized before the GITHUB_OUTPUT write.

Fix: Apply `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` immediately before every write to `$GITHUB_OUTPUT` when the value originates from user-controlled input.

Locations:

- `action.yml:34`
- `action.yml:47`

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

Fixed all script injection and github-env-injection findings in action.yml:

1. **Set parameters step**: Moved INPUT_COMMENT, INPUT_CLOSE_REASON, INPUT_LABELS to env: block. Replaced all ${{ inputs.* }} inline expressions with $VAR_NAME references. Added safe_labels sanitization (tr -d '\n\r') before writing labels to $GITHUB_OUTPUT. The comment output uses heredoc delimiter form (comment<<$delimiter) which already prevents injection per the rules.

2. **Close Issue step**: Moved INPUT_REPOSITORY, INPUT_ISSUE_NUMBER, PARAMS_CLOSE_REASON, PARAMS_COMMENT to env: block. The PARAMS_COMMENT value (which contains '--comment "..."') is tokenized using the xargs idiom into an args array to preserve argument boundaries.

3. **Add Labels step**: Moved INPUT_REPOSITORY, INPUT_ISSUE_NUMBER, PARAMS_LABELS to env: block. All ${{ }} expressions removed from run: blocks.

