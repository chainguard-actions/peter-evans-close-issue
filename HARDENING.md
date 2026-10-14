<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-issue/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-issue/v3.0.1** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings across all three composite steps, allowing an attacker-controlled input to inject arbitrary shell commands.

**Step "Set parameters" (lines 31–46):**
- Line 31: `if [ -n "${{ inputs.comment }}" ]; then`
- Line 32: `comment="--comment \"${{ inputs.comment }}\""`
- Line 39: `if [ "${{ inputs.close-reason }}" == "not_planned" ]; then`
- Line 45: `if [ -n "${{ inputs.labels }}" ]; then`
- Line 46: `labels=$(echo "${{ inputs.labels }}" | tr '\n' ',' | sed 's/,$//')`

**Step "Close Issue" (lines 53–56):**
- Line 53: `gh issue close -R "${{ inputs.repository }}"`
- Line 54: `--reason "${{ steps.params.outputs.close-reason }}"`
- Line 55: `${{ steps.params.outputs.comment }}` (also unquoted)
- Line 56: `"${{ inputs.issue-number }}"`

**Step "Add Labels" (lines 63–65):**
- Line 63: `gh issue edit -R "${{ inputs.repository }}"`
- Line 64: `--add-label "${{ steps.params.outputs.labels }}"`
- Line 65: `"${{ inputs.issue-number }}"`

All inputs (`inputs.comment`, `inputs.close-reason`, `inputs.labels`, `inputs.repository`, `inputs.issue-number`) and step outputs (`steps.params.outputs.*`) are workflow-controllable and must be passed via `env:` variables and referenced as quoted `"$VAR"` shell variables instead of being interpolated directly.

Locations:

- `action.yml:31`
- `action.yml:32`
- `action.yml:39`
- `action.yml:45`
- `action.yml:46`
- `action.yml:53`
- `action.yml:54`
- `action.yml:55`
- `action.yml:56`
- `action.yml:63`
- `action.yml:64`
- `action.yml:65`

### github-env-injection (severity: high)

The "Set parameters" step writes values derived from untrusted `inputs.*` to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled newline character in the input can inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent outputs or poisoning the step-output context.

**Finding 1 — `inputs.comment` → `$GITHUB_OUTPUT` (line 35):**
`inputs.comment` is interpolated into the shell variable `comment` (line 32) and then written unsanitized:
```
echo "$comment" >> $GITHUB_OUTPUT
```
A value containing a newline (e.g. `foo\nmalicious-key=injected`) will inject an extra key into `$GITHUB_OUTPUT`.

**Finding 2 — `inputs.labels` → `$GITHUB_OUTPUT` (line 49):**
`inputs.labels` is processed into the shell variable `labels` (line 46) and then written unsanitized:
```
echo labels=$labels >> $GITHUB_OUTPUT
```
The `tr '\n' ','` transformation only converts newlines to commas within the pipeline; the final `$labels` value is still written without `tr -d '\n\r'` sanitization before the `>> $GITHUB_OUTPUT` write.

Fix: apply `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` immediately before each write to `$GITHUB_OUTPUT`.

Locations:

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

Fixed all script-injection and github-env-injection findings in action.yml:

1. **Set parameters step**: Moved `inputs.comment`, `inputs.close-reason`, and `inputs.labels` to `env:` block as `INPUT_COMMENT`, `INPUT_CLOSE_REASON`, `INPUT_LABELS`. Updated shell script to reference these env vars. Added `safe_labels=$(printf '%s' "$labels" | tr -d '\n\r')` before writing labels to $GITHUB_OUTPUT. The comment output uses heredoc delimiter form (already safe for multi-line values per the rules).

2. **Close Issue step**: Moved `inputs.repository`, `inputs.issue-number`, `steps.params.outputs.close-reason`, and `steps.params.outputs.comment` to `env:` block. The `PARAMS_COMMENT` value (which contains `--comment "..."`) is tokenized via xargs into a bash array to preserve argument boundaries.

3. **Add Labels step**: Moved `inputs.repository`, `inputs.issue-number`, and `steps.params.outputs.labels` to `env:` block. Shell script references these as quoted env vars.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in action.yml at line 35. Added sanitization of INPUT_COMMENT before embedding it in the comment variable: `safe_comment=$(printf '%s' "$INPUT_COMMENT" | tr -d '\n\r')`. The `comment` variable now uses `$safe_comment` instead of `$INPUT_COMMENT`, stripping all newlines and carriage returns from the user-controlled input before it's written to $GITHUB_OUTPUT. The heredoc delimiter form is preserved as it correctly handles the key boundary.

