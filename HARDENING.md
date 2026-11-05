<!-- markdownlint-disable -->

# Hardening Report: pre-commit--action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pre-commit--action/v3.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a tag instead of a full 40-character commit SHA: `uses: actions/cache@v4`.

Locations:

- `action.yml:15`

### script-injection (severity: high)

Sub-rule (a): an expression is placed directly in a run: block: `run: pre-commit run --show-diff-on-failure --color=always ${{ inputs.extra_args }}`. Whoever calls the action controls inputs.extra_args, and the value is not quoted, so shell metacharacters in it can run commands. Fix: pass the value in through env (e.g. EXTRA_ARGS) and expand it safely.

Locations:

- `action.yml:18`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.extra_args }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:20`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 18ded0d..f31817f 100644
--- a/action.yml
+++ b/action.yml
@@ -8,13 +8,38 @@ inputs:
 runs:
   using: composite
   steps:
-  - run: python -m pip install pre-commit
-    shell: bash
-  - run: python -m pip freeze --local
-    shell: bash
   - uses: actions/cache@v4
-    with:
-      path: ~/.cache/pre-commit
-      key: pre-commit-3|${{ env.pythonLocation }}|${{ hashFiles('.pre-commit-config.yaml') }}
-  - run: pre-commit run --show-diff-on-failure --color=always ${{ inputs.extra_args }}
-    shell: bash
+    - run: python -m pip install pre-commit
+      shell: bash
+    - run: python -m pip freeze --local
+      shell: bash
+      with:
+        path: ~/.cache/pre-commit
+        key: pre-commit-3|${{ env.pythonLocation }}|${{ hashFiles('.pre-commit-config.yaml') }}
+    - env:
+        EXTRA_ARGS: ${{ inputs.extra_args }}
+      run: |
+        args=()
+        if [ -n "$EXTRA_ARGS" ]; then
+          while IFS= read -r -d '' t; do args+=("$t"); done \
+            < <(printf '%s' "$EXTRA_ARGS" | xargs printf '%s\0')
+        fi
+        pre-commit run --show-diff-on-failure --color=always "${args[@]}"
+      shell: bash

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, prior-fix

**Notes:**

Pinned actions/cache to 0057852bfaa89a56745cba8c7296529d2fc39830 (v4). Moved inputs.extra_args into env EXTRA_ARGS and tokenized it quote-aware (xargs -r, with unmatched-quote check and empty guard) into a bash array, matching upstream's inline-expression splitting. Re-applied prior fix but with correct YAML step structure (the previous patch had mis-nested steps).

