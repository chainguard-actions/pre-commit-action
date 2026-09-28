<!-- markdownlint-disable -->

# Hardening Report: pre-commit--action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pre-commit--action/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block directly interpolates `${{ inputs.extra_args }}` into the shell command string: `pre-commit run --show-diff-on-failure --color=always ${{ inputs.extra_args }}`. Because GitHub Actions performs template substitution before the shell parses the command, an attacker can supply a value like `; malicious-command` via the `extra_args` input to execute arbitrary shell commands. Fix: move the input into an `env:` variable and reference it as a quoted shell variable, e.g. `env: EXTRA_ARGS: ${{ inputs.extra_args }}` and then `pre-commit run --show-diff-on-failure --color=always "$EXTRA_ARGS"`.

Locations:

- `action.yml:14`

### unpinned-uses (severity: high)

The composite action step `uses: actions/cache@v4` references a mutable tag (`@v4`) rather than a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack. Pin to a specific SHA, e.g. `uses: actions/cache@5a3ec84eff668545956fd18022155c47e93e2684 # v4`.

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

Fixed three findings in hardened/action/action.yml: (1) Pinned actions/cache@v4 to full SHA 0057852bfaa89a56745cba8c7296529d2fc39830 with the tag preserved as a comment. (2) & (3) Moved ${{ inputs.extra_args }} out of the run: block into an env: variable (EXTRA_ARGS). Since extra_args is a list-style input (options/flags for pre-commit), used the xargs-based tokenization idiom to split it into an array while preserving quoted arguments, then expanded it as "${args[@]}" to pre-commit run.

