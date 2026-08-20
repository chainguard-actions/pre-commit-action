<!-- markdownlint-disable -->

# Hardening Report: pre-commit--action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pre-commit--action/v3.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The `run:` block on line 18 of action.yml directly interpolates `${{ inputs.extra_args }}` into the shell command string: `pre-commit run --show-diff-on-failure --color=always ${{ inputs.extra_args }}`. Because `inputs.extra_args` is caller-controlled, an attacker can supply shell metacharacters (e.g. `; malicious-command`) that will be executed by the shell before any quoting can protect them. The fix is to pass the value via an `env:` variable and double-quote it in the script: `env:\n  EXTRA_ARGS: ${{ inputs.extra_args }}\nrun: pre-commit run --show-diff-on-failure --color=always "$EXTRA_ARGS"`.

Locations:

- `action.yml:18`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or hijacked.

- action.yml line 13: `uses: actions/cache@v4` (tag `v4`)
- .github/workflows/main.yml line 9: `uses: actions/checkout@v3` (tag `v3`)
- .github/workflows/main.yml line 10: `uses: actions/setup-python@v3` (tag `v3`)

Each should be pinned to a full SHA, e.g. `actions/cache@0c45773b623bea8c8e75f6c82b208c3cf94ea4f # v4`.

Locations:

- `action.yml:13`
- `.github/workflows/main.yml:9`
- `.github/workflows/main.yml:10`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/main.yml` has no top-level `permissions:` key and the single job `main` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all` for older repositories), granting unnecessary write access. A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level or on the job.

Locations:

- `.github/workflows/main.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.extra_args }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings:
1. script-injection / static-inline-injection (action.yml line 18/20): Moved `${{ inputs.extra_args }}` into an `env:` block as EXTRA_ARGS, then tokenized it with xargs into a bash array before passing to pre-commit. This safely handles the list-style input without shell injection.
2. unpinned-uses (action.yml line 13): Pinned actions/cache@v4 → @0057852bfaa89a56745cba8c7296529d2fc39830 # v4.
3. unpinned-uses (.github/workflows/main.yml lines 9-10): Pinned actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3 and actions/setup-python@v3 → @3542bca2639a428e1796aaa6a2ffef0c0f575566 # v3.
4. missing-permissions (.github/workflows/main.yml): Added top-level `permissions: contents: read` block.

