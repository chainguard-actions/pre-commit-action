<!-- markdownlint-disable -->

# Hardening Report: pre-commit--action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pre-commit--action/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The expression `${{ inputs.extra_args }}` is interpolated directly inside a `run:` shell command string. `inputs.extra_args` is caller-controlled and can contain shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.), enabling arbitrary command injection. The offending line is: `pre-commit run --show-diff-on-failure --color=always ${{ inputs.extra_args }}`. Fix: move the value into an `env:` variable and double-quote it in the shell script, e.g. `env: EXTRA_ARGS: ${{ inputs.extra_args }}` and then `pre-commit run --show-diff-on-failure --color=always "$EXTRA_ARGS"`.

Locations:

- `action.yml:14`

### unpinned-uses (severity: high)

The step `uses: actions/cache@v4` references a mutable tag (`v4`) rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, creating a supply-chain risk. Pin to a full SHA, e.g. `uses: actions/cache@5a3ec84eff668545956fd18022155c47e93e2684 # v4`.

Locations:

- `action.yml:11`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.extra_args }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed three findings in hardened/action/action.yml:
1. Pinned `actions/cache@v4` to full SHA `0057852bfaa89a56745cba8c7296529d2fc39830` (keeping `# v4` comment for readability).
2. Moved `${{ inputs.extra_args }}` out of the `run:` shell string into an `env:` block as `EXTRA_ARGS`. Since `extra_args` is a list of CLI options, used the xargs-based tokenization pattern to split it into a bash array (`args=()`), then expanded `"${args[@]}"` when calling `pre-commit run`. This prevents shell injection while correctly handling quoted arguments like `sh -c "exit 0"` in the input.

