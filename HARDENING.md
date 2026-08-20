<!-- markdownlint-disable -->

# Hardening Report: dorny--paths-filter/v4.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dorny--paths-filter/v4.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files use action references pinned to mutable version tags (@v6) instead of immutable 40-character SHA digests. This exposes the workflow to supply-chain attacks if the tag is moved. Affected references: actions/checkout@v6, actions/setup-node@v6.

Locations:

- `.github/workflows/build.yml:11`
- `.github/workflows/build.yml:12`
- `.github/workflows/build.yml:17`
- `.github/workflows/pull-request-verification.yml:13`
- `.github/workflows/pull-request-verification.yml:14`
- `.github/workflows/pull-request-verification.yml:30`
- `.github/workflows/pull-request-verification.yml:47`
- `.github/workflows/pull-request-verification.yml:56`
- `.github/workflows/pull-request-verification.yml:65`
- `.github/workflows/pull-request-verification.yml:77`
- `.github/workflows/pull-request-verification.yml:91`

### script-injection (severity: high)

Rule (a): Three run: steps in the test-change-type job directly interpolate ${{steps.filter.outputs.*}} expressions inside shell commands. The values of steps.*.outputs.* flow through YAML template substitution before the shell sees them, allowing an attacker who can influence those output values to inject arbitrary shell commands. Offending lines: `run: echo ${{steps.filter.outputs.added_files}}`, `run: echo ${{steps.filter.outputs.modified_files}}`, `run: echo ${{steps.filter.outputs.deleted_files}}`. Fix: move the values into env: variables and reference them as quoted shell variables, e.g. `echo "$ADDED_FILES"`.

Locations:

- `.github/workflows/pull-request-verification.yml:109`
- `.github/workflows/pull-request-verification.yml:111`
- `.github/workflows/pull-request-verification.yml:113`

### missing-permissions (severity: medium)

Neither workflow file has a top-level `permissions:` block, and several jobs also lack job-level permissions. In build.yml, neither the `build` nor the `self-test` job declares permissions. In pull-request-verification.yml, the jobs `build`, `test-without-token`, `test-wd-without-token`, `test-local-changes`, and `test-change-type` have no permissions block (only `test-inline` and `test-external` do). Without explicit permissions, jobs inherit the repository's default token permissions, which may be overly broad. Add `permissions: {}` or minimal specific scopes to each job (or a restrictive top-level block).

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/pull-request-verification.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned all `actions/checkout@v6` to SHA `d23441a48e516b6c34aea4fa41551a30e30af803` and all `actions/setup-node@v6` to SHA `249970729cb0ef3589644e2896645e5dc5ba9c38` in both `.github/workflows/build.yml` and `.github/workflows/pull-request-verification.yml`. Original tag preserved as inline comment (# v6).

2. **script-injection**: Fixed three `run: echo ${{steps.filter.outputs.*}}` steps in the `test-change-type` job by moving each expression into an `env:` block (ADDED_FILES, MODIFIED_FILES, DELETED_FILES) and referencing them as quoted shell variables in the run command.

3. **missing-permissions**: Added `permissions: {}` to all jobs lacking a permissions block — `build` and `self-test` in build.yml, and `build`, `test-without-token`, `test-wd-without-token`, `test-local-changes`, and `test-change-type` in pull-request-verification.yml. The `test-inline` and `test-external` jobs already had appropriate `pull-requests: read` permissions and were left unchanged.

