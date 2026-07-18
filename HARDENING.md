<!-- markdownlint-disable -->

# Hardening Report: dorny--paths-filter/v4.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dorny--paths-filter/v4.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag-based refs (`@v6`) instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the upstream action tag is moved or compromised. Affected references: `actions/checkout@v6` and `actions/setup-node@v6` in both workflow files.

Locations:

- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:22`
- `.github/workflows/pull-request-verification.yml:13`
- `.github/workflows/pull-request-verification.yml:14`
- `.github/workflows/pull-request-verification.yml:33`
- `.github/workflows/pull-request-verification.yml:52`
- `.github/workflows/pull-request-verification.yml:63`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside `run:` shell command strings (sub-rule a). GitHub Actions template substitution occurs before the shell parses the command, so even single-quoted strings are not safe. (1) `echo '${{ matrix.locale }} UTF-8' >> /etc/locale.gen` — `${{ matrix.locale }}` is a matrix context value injected directly into a shell command. (2) Three `run: echo ${{steps.filter.outputs.*}}` lines — `steps.*.outputs.*` values are interpolated directly into shell commands without quoting or sanitization.

Locations:

- `.github/workflows/pull-request-verification.yml:83`
- `.github/workflows/pull-request-verification.yml:155`
- `.github/workflows/pull-request-verification.yml:157`
- `.github/workflows/pull-request-verification.yml:159`

### missing-permissions (severity: medium)

Neither workflow file has a top-level `permissions:` block, and several jobs also lack job-level `permissions:` blocks. Without explicit permissions, GitHub Actions grants the default token permissions (which may include `contents: write` on some repository configurations), violating the principle of least privilege. In `build.yml`, neither the `build` nor `self-test` job has permissions defined. In `pull-request-verification.yml`, the `build`, `test-without-token`, `test-container-without-token`, `test-wd-without-token`, `test-local-changes`, and `test-change-type` jobs all lack `permissions:` blocks.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/pull-request-verification.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in both workflow files:

1. **unpinned-uses**: Pinned all `actions/checkout@v6` → `@df4cb1c069e1874edd31b4311f1884172cec0e10 # v6` and `actions/setup-node@v6` → `@249970729cb0ef3589644e2896645e5dc5ba9c38 # v6` in both build.yml and pull-request-verification.yml.

2. **script-injection**: (a) Moved `${{ matrix.locale }}` out of the `echo '...' >> /etc/locale.gen` shell command into an `env: MATRIX_LOCALE:` block, using `echo "$MATRIX_LOCALE UTF-8"` in the run script. (b) Moved the three `${{steps.filter.outputs.added_files}}`, `${{steps.filter.outputs.modified_files}}`, and `${{steps.filter.outputs.deleted_files}}` expressions out of `run:` strings into `env:` blocks (`ADDED_FILES`, `MODIFIED_FILES`, `DELETED_FILES`), referencing them as `echo "$VAR"` in the shell.

3. **missing-permissions**: Added `permissions: {}` at the workflow top level in both files. Added `permissions: { contents: read }` to all jobs that lacked permissions (build, self-test, test-without-token, test-container-without-token, test-wd-without-token, test-local-changes, test-change-type). Preserved existing `pull-requests: read` on test-inline and test-external, adding `contents: read` to those as well.

