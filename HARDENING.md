<!-- markdownlint-disable -->

# Hardening Report: WyriHaximus--github-action-get-previous-tag/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **WyriHaximus--github-action-get-previous-tag/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in ci.yml directly interpolate ${{ steps.*.outputs.tag }} and ${{ steps.*.outputs.timestamp }} expressions inside shell commands (sub-rule a). These steps.*.outputs.* values flow through YAML template substitution before the shell processes them, allowing an attacker who can influence step outputs to inject arbitrary shell commands. Offending lines include: `echo "Tag: ${{ steps.previoustag.outputs.tag }}"`, `test -n "${{ steps.previoustag.outputs.tag }}"`, and similar patterns across all four jobs. The values should be passed via env: variables and then referenced as quoted shell variables (e.g. "$TAG") instead.

Locations:

- `.github/workflows/ci.yml:33`
- `.github/workflows/ci.yml:60`
- `.github/workflows/ci.yml:78`
- `.github/workflows/ci.yml:92`
- `.github/workflows/ci.yml:151`
- `.github/workflows/ci.yml:179`

### unpinned-uses (severity: high)

Several uses: references are pinned to mutable tags or branch names rather than immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if those refs are moved or compromised. Failing references: (1) ci.yml — `therussiankid92/gat@v1` appears 6 times (a mutable tag); (2) release-management.yaml — `WyriHaximus/github-workflows/.github/workflows/github-action-release-management.yaml@main` (a mutable branch). All should be pinned to full SHA digests.

Locations:

- `.github/workflows/ci.yml:37`
- `.github/workflows/ci.yml:64`
- `.github/workflows/ci.yml:83`
- `.github/workflows/ci.yml:96`
- `.github/workflows/ci.yml:155`
- `.github/workflows/ci.yml:183`
- `.github/workflows/release-management.yaml:12`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level permissions: key and none of its four jobs (get-previous-tag-with-working-directory, get-previous-tag, get-previous-tag-on-github-action-with-expected-order, monotonic-release-tag) define a job-level permissions: block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to all scopes). A minimal permissions: block (e.g. contents: read) should be added at the top level or on each job.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings in ci.yml and release-management.yaml:

1. script-injection: Moved all ${{ steps.*.outputs.tag }} and ${{ steps.*.outputs.timestamp }} expressions out of run: shell blocks and into env: blocks (TAG and TIMESTAMP variables), then referenced them as $TAG and $TIMESTAMP in the shell scripts. This affects 4 jobs with a total of 6 run: blocks.

2. unpinned-uses: Pinned all 6 occurrences of therussiankid92/gat@v1 in ci.yml to SHA 2c6f703798d3b670c6e1128e1f9398d0b2510e08 (# v1). Pinned WyriHaximus/github-workflows/...@main in release-management.yaml to SHA eccae02a0bb5f0a5ab38b3207611aea5ca041321 (# main).

3. missing-permissions: Added top-level 'permissions: contents: read' block to ci.yml, providing the minimum necessary permissions for a CI workflow that only reads repository content.

