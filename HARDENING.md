<!-- markdownlint-disable -->

# Hardening Report: pre-commit--action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pre-commit--action/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The expression `${{ inputs.extra_args }}` is interpolated directly inside a `run:` shell command string on line 15. An attacker (or any caller of this composite action) can supply a value containing shell metacharacters (`;`, `|`, `$(...)`, etc.) that will be executed by the shell before any quoting can protect them. The value should be passed via an `env:` variable and then referenced as a double-quoted shell variable, e.g.: `env:\n  EXTRA_ARGS: ${{ inputs.extra_args }}\nrun: pre-commit run --show-diff-on-failure --color=always "$EXTRA_ARGS"`

Locations:

- `action.yml:15`

### unpinned-uses (severity: high)

The step `uses: actions/cache@v4` references a mutable tag (`@v4`) rather than a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack. Pin to a specific commit SHA, e.g. `actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v4`.

Locations:

- `action.yml:12`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.extra_args }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed three findings in hardened/action/action.yml:
1. Pinned `actions/cache@v4` to full commit SHA `0057852bfaa89a56745cba8c7296529d2fc39830` (keeping `# v4` comment for readability).
2. Moved `${{ inputs.extra_args }}` out of the `run:` shell string into an `env:` block as `EXTRA_ARGS`. Since `extra_args` is a list-style input (space-separated options/flags), used the xargs tokenization pattern with a bash array to safely split the value into individual arguments while preserving quoting, then passed `"${args[@]}"` to `pre-commit run`.

