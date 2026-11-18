<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-issue/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-issue/v3.0.1** was hardened automatically. 14 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Set parameters' step puts ${{ }} expressions straight into the run: script, for example `if [ -n "${{ inputs.comment }}" ]; then`, `comment="--comment \"${{ inputs.comment }}\""`, `if [ "${{ inputs.close-reason }}" == "not_planned" ]`, and `labels=$(echo "${{ inputs.labels }}" | ...)`. Rule (b): it also expands workflow-derived data without quotes, as in `labels=$(echo $labels | ...)` and `echo labels=$labels`. An attacker-controlled comment or label value can break out of the quotes and run arbitrary commands.

Locations:

- `action.yml:32`

### script-injection (severity: high)

Rule (a): the 'Close Issue' step puts ${{ inputs.repository }}, ${{ steps.params.outputs.close-reason }}, ${{ steps.params.outputs.comment }} (unquoted) and ${{ inputs.issue-number }} straight into the run: script. The comment output comes from inputs.comment, so the template substitution allows command injection.

Locations:

- `action.yml:55`

### script-injection (severity: high)

Rule (a): the 'Add Labels' step puts ${{ inputs.repository }}, ${{ steps.params.outputs.labels }} and ${{ inputs.issue-number }} straight into the run: script.

Locations:

- `action.yml:66`

### github-env-injection (severity: high)

The 'Set parameters' step builds a value from inputs.labels and writes it to $GITHUB_OUTPUT with `echo labels=$labels >> $GITHUB_OUTPUT`. The required `printf '%s' ... | tr -d '\n\r'` step does not run right before the write. (The comment output passes because it is written between delimiters from openssl rand.)

Locations:

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index d097152..934f4c4 100644
--- a/action.yml
+++ b/action.yml
@@ -26,52 +26,79 @@ inputs:
 runs:
   using: composite
   steps:
     - name: Set parameters
       id: params
       shell: bash
+      env:
+        INPUT_COMMENT: ${{ inputs.comment }}
+        INPUT_CLOSE_REASON: ${{ inputs.close-reason }}
+        INPUT_LABELS: ${{ inputs.labels }}
       run: |
-        if [ -n "${{ inputs.comment }}" ]; then
-          comment="--comment \"${{ inputs.comment }}\""
-          delimiter="$(openssl rand -hex 8)"
-          echo "comment<<$delimiter" >> $GITHUB_OUTPUT
-          echo "$comment" >> $GITHUB_OUTPUT
-          echo "$delimiter" >> $GITHUB_OUTPUT
+        if [ -n "$INPUT_COMMENT" ]; then
+          safe_comment=$(printf '%s' "$INPUT_COMMENT" | tr -d '\n\r')
+          echo "comment=$safe_comment" >> $GITHUB_OUTPUT
         fi
 
-        if [ "${{ inputs.close-reason }}" == "not_planned" ]; then
+        if [ "$INPUT_CLOSE_REASON" == "not_planned" ]; then
           echo close-reason="not planned" >> $GITHUB_OUTPUT
         else
           echo close-reason="completed" >> $GITHUB_OUTPUT
         fi
 
         # Convert labels to comma separated list
-        if [ -n "${{ inputs.labels }}" ]; then
-          labels=$(echo "${{ inputs.labels }}" | tr '\n' ',' | sed 's/,$//')
+        if [ -n "$INPUT_LABELS" ]; then
+          labels=$(printf '%s' "$INPUT_LABELS" | tr '\n' ',' | sed 's/,$//')
           # Remove trailing comma and whitespace
-          labels=$(echo $labels | sed 's/,$//' | sed 's/[[:space:]]//g')
-          echo labels=$labels >> $GITHUB_OUTPUT
+          labels=$(printf '%s' "$labels" | sed 's/,$//' | sed 's/[[:space:]]//g')
+          safe_labels=$(printf '%s' "$labels" | tr -d '\n\r')
+          echo "labels=$safe_labels" >> $GITHUB_OUTPUT
         fi
-
     - name: Close Issue
       shell: bash
-      run: |
-        gh issue close -R "${{ inputs.repository }}" \
-        --reason "${{ steps.params.outputs.close-reason }}" \
-        ${{ steps.params.outputs.comment }} \
-        "${{ inputs.issue-number }}"
       env:
         GH_TOKEN: ${{ inputs.token }}
-
+        INPUT_REPOSITORY: ${{ inputs.repository }}
+        INPUT_ISSUE_NUMBER: ${{ inputs.issue-number }}
+        PARAMS_CLOSE_REASON: ${{ steps.params.outputs.close-reason }}
+        PARAMS_COMMENT: ${{ steps.params.outputs.comment }}
+      run: |
+        args=()
+        if [ -n "$PARAMS_COMMENT" ]; then
+          args+=(--comment "$PARAMS_COMMENT")
+        fi
+        gh issue close -R "$INPUT_REPOSITORY" \
+          --reason "$PARAMS_CLOSE_REASON" \
+          "${args[@]}" \
+          "$INPUT_ISSUE_NUMBER"
     - name: Add Labels
       if: steps.params.outputs.labels != ''
       shell: bash
-      run: |
-        gh issue edit -R "${{ inputs.repository }}" \
-        --add-label "${{ steps.params.outputs.labels }}" \
-        "${{ inputs.issue-number }}"
       env:
         GH_TOKEN: ${{ inputs.token }}
-
+        INPUT_REPOSITORY: ${{ inputs.repository }}
+        INPUT_ISSUE_NUMBER: ${{ inputs.issue-number }}
+        PARAMS_LABELS: ${{ steps.params.outputs.labels }}
+      run: |
+        gh issue edit -R "$INPUT_REPOSITORY" \
+          --add-label "$PARAMS_LABELS" \
+          "$INPUT_ISSUE_NUMBER"
 branding:
   icon: 'git-pull-request'
   color: 'gray-dark'

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, prior-fix

**Notes:**

action.yml: every ${{ }} expression is now in step env: blocks, and the scripts read them as quoted shell variables. The comment is still written to GITHUB_OUTPUT between delimiters from openssl rand, so multi-line comments survive. It is now the raw comment text instead of a shell fragment built as --comment "...". The Close Issue step builds --comment with a bash array, so the flag and its value stay separate arguments and the value is never parsed as shell code. The labels value has newlines and carriage returns stripped with printf | tr -d '\n\r' right before it is written. The step also no longer expands $labels unquoted. Close reason, repository and issue number are passed as quoted env vars. The prior harden's changes are covered. Unlike that patch, this keeps the multi-line comment form instead of stripping newlines from the comment.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Values are no longer passed between steps through $GITHUB_OUTPUT. The comment input now goes straight to 'gh issue close' through env as one argument (args+=(--comment "$INPUT_COMMENT")). The labels list is now built inside the Add Labels step from inputs.labels, using upstream's conversion (newlines become commas, trailing comma and whitespace removed), and is passed straight to 'gh issue edit --add-label'. That step runs only when inputs.labels != ''. This removes the safe_labels output and the comment heredoc output. The close-reason output is unchanged. I did not run the action or re-run the checks.

