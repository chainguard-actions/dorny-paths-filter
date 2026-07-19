<!-- markdownlint-disable -->

# Hardening Report: dorny--paths-filter/v3.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dorny--paths-filter/v3.0.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in both workflow files use mutable tag-based refs (`@v4`) instead of immutable 40-character SHA commit digests. This exposes the workflow to supply-chain attacks if the referenced action tag is moved or compromised. Failing references include: `actions/checkout@v4`, `actions/setup-node@v4` in both files.

Locations:

- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:21`
- `.github/workflows/pull-request-verification.yml:13`
- `.github/workflows/pull-request-verification.yml:14`
- `.github/workflows/pull-request-verification.yml:26`
- `.github/workflows/pull-request-verification.yml:42`
- `.github/workflows/pull-request-verification.yml:55`
- `.github/workflows/pull-request-verification.yml:65`
- `.github/workflows/pull-request-verification.yml:77`
- `.github/workflows/pull-request-verification.yml:90`

### script-injection (severity: high)

Sub-rule (a): Three `run:` steps in the `test-change-type` job directly interpolate `${{steps.filter.outputs.*}}` expressions inside shell commands. GitHub Actions performs template substitution before the shell sees the string, so a malicious value in a step output could inject arbitrary shell commands. Offending lines: `run: echo ${{steps.filter.outputs.added_files}}`, `run: echo ${{steps.filter.outputs.modified_files}}`, `run: echo ${{steps.filter.outputs.deleted_files}}`.

Locations:

- `.github/workflows/pull-request-verification.yml:107`
- `.github/workflows/pull-request-verification.yml:109`
- `.github/workflows/pull-request-verification.yml:111`

### missing-permissions (severity: medium)

build.yml has no top-level `permissions:` key and none of its jobs (`build`, `self-test`) define job-level permissions, meaning the workflow runs with the default (potentially broad) GITHUB_TOKEN permissions. pull-request-verification.yml also has no top-level `permissions:` key, and the jobs `build`, `test-without-token`, `test-wd-without-token`, `test-local-changes`, and `test-change-type` all lack job-level `permissions:` blocks.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/pull-request-verification.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across both workflow files: (1) Pinned actions/checkout@v4 to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020 in both files; (2) Added top-level 'permissions: {}' to both files and job-level 'permissions: contents: read' to all jobs that lacked permissions; (3) Fixed script injection in the three 'Print' steps of test-change-type job by moving ${{ steps.filter.outputs.* }} expressions into env: blocks and referencing them as $ADDED_FILES, $MODIFIED_FILES, $DELETED_FILES in the shell commands.

