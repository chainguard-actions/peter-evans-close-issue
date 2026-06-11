<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-issue/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **peter-evans--close-issue/v3.0.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set parameters' step of action.yml, the `labels` output variable is derived from `inputs.labels` (an untrusted input mapped via `env: LABELS: ${{ inputs.labels }}`) and written to $GITHUB_OUTPUT without the required sanitization. The script uses `tr '\n' ','` to replace newlines with commas, but this does NOT strip carriage returns (`\r`). An attacker-controlled value containing `\r` could inject additional key=value pairs into $GITHUB_OUTPUT. The required sanitization pattern (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before the write: `echo labels="$labels" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:50`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in the 'Set parameters' step of action.yml (line 50). The original code used `tr '\n' ','` to process labels but did not strip carriage returns (`\r`), allowing an attacker to inject additional key=value pairs into $GITHUB_OUTPUT. The fix: (1) adds `tr -d '\r'` at each intermediate processing step using `printf '%s'` instead of `echo`, (2) introduces a final `safe_labels=$(printf '%s' "$labels" | tr -d '\n\r')` sanitization step before writing to $GITHUB_OUTPUT, and (3) uses the sanitized variable in the echo statement with proper quoting.

