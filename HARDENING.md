<!-- markdownlint-disable -->

# Hardening Report: pre-commit--action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pre-commit--action/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block on line 16 directly interpolates `${{ inputs.extra_args }}` into the shell command `pre-commit run --show-diff-on-failure --color=always ${{ inputs.extra_args }}`. The Actions runner substitutes this expression before the shell parses the command, so any caller who supplies a crafted `extra_args` value (e.g. `; malicious-command`) can execute arbitrary shell commands on the runner.

Locations:

- `action.yml:16`

### unpinned-uses (severity: high)

The composite action step `uses: actions/cache@v4` (line 11) references a mutable tag (`v4`) rather than a pinned 40-character commit SHA. If the `actions/cache` repository is compromised or the tag is moved, the action will silently execute attacker-controlled code. Pin to a full SHA, e.g. `actions/cache@5a3ec84eff668545956fd18022155c47e93e2684 # v4`.

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

1. Pinned `actions/cache@v4` to full SHA `0057852bfaa89a56745cba8c7296529d2fc39830` (tag preserved as comment). 2. Fixed script injection for `inputs.extra_args` (both findings refer to the same expression): moved it to an `env:` block as `EXTRA_ARGS`, then used xargs-based quote-aware tokenization into a bash array before passing to `pre-commit run --show-diff-on-failure --color=always`. This correctly handles the list/args-style input without collapsing it into a single argument.

