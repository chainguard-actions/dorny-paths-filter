<!-- markdownlint-disable -->

# Hardening Report: dorny--paths-filter/v3.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dorny--paths-filter/v3.0.3** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple uses: references in build.yml use mutable version tags (@v4) instead of pinned 40-character SHA commit hashes. Affected references: actions/checkout@v4 (lines 13, 24), actions/setup-node@v4 (line 14).

Locations:

- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:24`

### unpinned-uses (severity: high)

Multiple uses: references in pull-request-verification.yml use mutable version tags (@v4) instead of pinned 40-character SHA commit hashes. Affected references: actions/checkout@v4 (lines 13, 27, 48, 60, 73, 89, 109), actions/setup-node@v4 (line 14).

Locations:

- `.github/workflows/pull-request-verification.yml:13`
- `.github/workflows/pull-request-verification.yml:14`
- `.github/workflows/pull-request-verification.yml:27`
- `.github/workflows/pull-request-verification.yml:48`
- `.github/workflows/pull-request-verification.yml:60`
- `.github/workflows/pull-request-verification.yml:73`
- `.github/workflows/pull-request-verification.yml:89`
- `.github/workflows/pull-request-verification.yml:109`

### permissions (severity: medium)

missing-permissions: build.yml has no top-level permissions: key and neither the 'build' job nor the 'self-test' job defines a job-level permissions: block. This grants the default (potentially broad) token permissions to all jobs.

Locations:

- `.github/workflows/build.yml:1`

### permissions (severity: medium)

missing-permissions: pull-request-verification.yml has no top-level permissions: key. Jobs 'build', 'test-without-token', 'test-wd-without-token', 'test-local-changes', and 'test-change-type' have no job-level permissions: block, granting them default (potentially broad) token permissions.

Locations:

- `.github/workflows/pull-request-verification.yml:1`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. The steps.filter.outputs.* values (file paths from git) are interpolated directly into shell commands without quoting or sanitization. An attacker could craft a filename containing shell metacharacters to achieve command injection. Offending lines: `run: echo ${{steps.filter.outputs.added_files}}` (line 131), `run: echo ${{steps.filter.outputs.modified_files}}` (line 133), `run: echo ${{steps.filter.outputs.deleted_files}}` (line 135).

Locations:

- `.github/workflows/pull-request-verification.yml:131`
- `.github/workflows/pull-request-verification.yml:133`
- `.github/workflows/pull-request-verification.yml:135`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all findings across both workflow files:

1. **unpinned-uses** (build.yml + pull-request-verification.yml): Pinned `actions/checkout@v4` to SHA `34e114876b0b11c390a56381ad16ebd13914f8d5` and `actions/setup-node@v4` to SHA `49933ea5288caeca8642d1e84afbd3f7d6820020` in all occurrences, with `# v4` comments for readability.

2. **permissions** (build.yml): Added top-level `permissions: {}` and job-level `permissions: { contents: read }` for both the `build` and `self-test` jobs.

3. **permissions** (pull-request-verification.yml): Added top-level `permissions: {}` and job-level permissions for all jobs — `contents: read` for all jobs, plus `pull-requests: read` for `test-inline` and `test-external` which already had that permission.

4. **script-injection** (pull-request-verification.yml): Moved the three `${{steps.filter.outputs.*}}` expressions from inline `run:` shell commands into `env:` blocks (`ADDED_FILES`, `MODIFIED_FILES`, `DELETED_FILES`) and referenced them as properly double-quoted environment variables (`echo "$ADDED_FILES"` etc.).

