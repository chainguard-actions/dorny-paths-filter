<!-- markdownlint-disable -->

# Hardening Report: dorny--paths-filter/v4.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dorny--paths-filter/v4.0.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference GitHub Actions using mutable tag-based refs (@v6) instead of immutable full 40-character SHA digests. This exposes the workflow to supply-chain attacks if the tag is moved. Affected references: `actions/checkout@v6` and `actions/setup-node@v6` in both build.yml and pull-request-verification.yml.

Locations:

- `.github/workflows/build.yml:10`
- `.github/workflows/build.yml:11`
- `.github/workflows/build.yml:17`
- `.github/workflows/pull-request-verification.yml:13`
- `.github/workflows/pull-request-verification.yml:14`
- `.github/workflows/pull-request-verification.yml:32`
- `.github/workflows/pull-request-verification.yml:47`
- `.github/workflows/pull-request-verification.yml:56`
- `.github/workflows/pull-request-verification.yml:80`
- `.github/workflows/pull-request-verification.yml:113`
- `.github/workflows/pull-request-verification.yml:122`
- `.github/workflows/pull-request-verification.yml:131`
- `.github/workflows/pull-request-verification.yml:157`

### script-injection (severity: high)

Sub-rule (a): The 'Generate locale' step in the `test-container-without-token` job directly interpolates `${{ matrix.locale }}` inside a `run:` shell command: `echo '${{ matrix.locale }} UTF-8' >> /etc/locale.gen`. The expression is substituted by the GitHub Actions template engine before the shell executes the command, allowing a matrix value containing shell metacharacters to be interpreted by the shell. All `${{ ... }}` expressions in run: blocks are findings regardless of their source context.

Locations:

- `.github/workflows/pull-request-verification.yml:88`

### missing-permissions (severity: medium)

Neither workflow file has a top-level `permissions:` key, and several jobs within them also lack job-level `permissions:` blocks. In build.yml, both jobs (`build` and `self-test`) have no permissions defined. In pull-request-verification.yml, the jobs `build`, `test-without-token`, `test-container-without-token`, `test-wd-without-token`, `test-local-changes`, `test-predicate-quantifier-some-with-excludes`, and `test-change-type` all lack permissions. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/pull-request-verification.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned `actions/checkout@v6` to SHA `d23441a48e516b6c34aea4fa41551a30e30af803` and `actions/setup-node@v6` to SHA `249970729cb0ef3589644e2896645e5dc5ba9c38` in both build.yml (3 occurrences) and pull-request-verification.yml (10 occurrences), preserving the tag as a comment.

2. **script-injection**: Fixed the 'Generate locale' step in `test-container-without-token` job by moving `${{ matrix.locale }}` into an `env:` block as `MATRIX_LOCALE` and referencing it as `"$MATRIX_LOCALE"` in the shell `run:` command.

3. **missing-permissions**: Added `permissions: {}` at the top level of both workflow files. Added `permissions: contents: read` to all jobs that lacked explicit permissions: `build` and `self-test` in build.yml; `build`, `test-without-token`, `test-container-without-token`, `test-wd-without-token`, `test-local-changes`, `test-predicate-quantifier-some-with-excludes`, and `test-change-type` in pull-request-verification.yml. Jobs `test-inline` and `test-external` already had `pull-requests: read` and were left unchanged.

