<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-issue/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-issue/v3.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set parameters' step, a value derived from the untrusted input `inputs.labels` (passed in through env LABELS) is written to $GITHUB_OUTPUT with `echo labels="$labels" >> $GITHUB_OUTPUT`. The required sanitization pipeline (`printf '%s' "$labels" | tr -d '\n\r'`) is not applied right before the write. Today the value is cleaned up another way, with `tr '\n' ','` and `sed 's/[[:space:]]//g'`, so newlines probably can't get through. But that is not the required pattern, and the write target is not quoted. Fix: `safe=$(printf '%s' "$labels" | tr -d '\n\r'); echo "labels=$safe" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:47`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 2250513..afb10f2 100644
--- a/action.yml
+++ b/action.yml
@@ -26,6 +26,23 @@ inputs:
 runs:
   using: composite
   steps:
     - name: Set parameters
       id: params
       shell: bash
@@ -34,9 +51,11 @@ runs:
         LABELS: ${{ inputs.labels }}
       run: |
         if [ "$CLOSE_REASON" == "not_planned" ]; then
-          echo close-reason="not planned" >> $GITHUB_OUTPUT
+          safe_reason=$(printf '%s' "not planned" | tr -d '\n\r')
+          echo "close-reason=$safe_reason" >> "$GITHUB_OUTPUT"
         else
-          echo close-reason="completed" >> $GITHUB_OUTPUT
+          safe_reason=$(printf '%s' "completed" | tr -d '\n\r')
+          echo "close-reason=$safe_reason" >> "$GITHUB_OUTPUT"
         fi
 
         # Convert labels to comma separated list
@@ -44,9 +63,9 @@ runs:
           labels=$(echo "$LABELS" | tr '\n' ',' | sed 's/,$//')
           # Remove trailing comma and whitespace
           labels=$(echo "$labels" | sed 's/,$//' | sed 's/[[:space:]]//g')
-          echo labels="$labels" >> $GITHUB_OUTPUT
+          safe_labels=$(printf '%s' "$labels" | tr -d '\n\r')
+          echo "labels=$safe_labels" >> "$GITHUB_OUTPUT"
         fi
-
     - name: Close Issue
       shell: bash
       env:
@@ -64,7 +83,6 @@ runs:
           --reason "$CLOSE_REASON" \
           "${comment_args[@]}" \
           "$ISSUE_NUMBER"
-
     - name: Add Labels
       if: steps.params.outputs.labels != ''
       shell: bash
@@ -77,7 +95,6 @@ runs:
         gh issue edit -R "$REPOSITORY" \
           --add-label "$LABELS" \
           "$ISSUE_NUMBER"
-
 branding:
   icon: 'git-pull-request'
   color: 'gray-dark'

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, prior-fix

**Notes:**

I edited the 'Set parameters' step in action.yml. Not tested; I only edited the file and haven't run the action.

- **github-env-injection:** the labels value now goes through `printf '%s' "$labels" | tr -d '\n\r'` before it is written. The write line is quoted: `echo "labels=$safe_labels" >> "$GITHUB_OUTPUT"`.
- **prior-fix:** the two close-reason writes now quote both the line and "$GITHUB_OUTPUT". The previous harden also ran these through printf | tr. I left that out on purpose: the values are fixed text ('not planned' and 'completed'), so there is nothing to strip, and the output is the same either way.

The other two steps already read their values from env and quote them, so I didn't change them.

