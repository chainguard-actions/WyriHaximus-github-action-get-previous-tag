<!-- markdownlint-disable -->

# Hardening Report: WyriHaximus--github-action-get-previous-tag/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **WyriHaximus--github-action-get-previous-tag/v1.2.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of immutable 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved.

.github/workflows/ci.yml:
  - uses: actions/checkout@v3 (line 19)
  - uses: JesseTG/rm@v1.0.2 (line 32)

.github/workflows/craft-release.yaml:
  - uses: WyriHaximus/github-action-wait-for-status@master (line 17)
  - uses: WyriHaximus/github-action-jwage-changelog-generator@master (line 33)
  - uses: actions/checkout@v3 (line 47)
  - uses: actions/create-release@v1 (line 71)
  - uses: haya14busa/action-update-semver@v1 (line 77)

.github/workflows/set-milestone-on-pr.yaml:
  - uses: actions/checkout@v3 (line 12)
  - uses: WyriHaximus/github-action-get-previous-tag@master (line 17)
  - uses: WyriHaximus/github-action-next-semvers@master (line 37)
  - uses: WyriHaximus/github-action-get-milestones@master (line 60)
  - uses: WyriHaximus/github-action-create-milestone@master (line 67)
  - uses: WyriHaximus/github-action-set-milestone@master (line 77)

Locations:

- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:32`
- `.github/workflows/craft-release.yaml:17`
- `.github/workflows/craft-release.yaml:33`
- `.github/workflows/craft-release.yaml:47`
- `.github/workflows/craft-release.yaml:71`
- `.github/workflows/craft-release.yaml:77`
- `.github/workflows/set-milestone-on-pr.yaml:12`
- `.github/workflows/set-milestone-on-pr.yaml:17`
- `.github/workflows/set-milestone-on-pr.yaml:37`
- `.github/workflows/set-milestone-on-pr.yaml:60`
- `.github/workflows/set-milestone-on-pr.yaml:67`
- `.github/workflows/set-milestone-on-pr.yaml:77`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual job within them defines job-level permissions either. Without explicit permissions, workflows run with the default (potentially broad) GITHUB_TOKEN permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/craft-release.yaml:1`
- `.github/workflows/set-milestone-on-pr.yaml:1`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings. Before the shell executes the command, GitHub Actions performs text substitution of the expression value, allowing an attacker who controls the value to inject arbitrary shell commands.

.github/workflows/ci.yml — sub-rule (a): `${{ steps.previoustag.outputs.tag }}` and `${{ steps.previoustag.outputs.timestamp }}` are interpolated directly in a `run:` block (lines 26–29). Same pattern repeated for `steps.previoustagwithfallback.outputs.*` (lines 39–42). Step outputs can be attacker-controlled via PR branch names or commit messages processed by the action.

.github/workflows/craft-release.yaml — sub-rule (a): `${{ env.MILESTONE }}` (which is set from `github.event.milestone.title`, an attacker-controllable value) is interpolated directly into `run:` shell commands:
  - Line 52: `run: git checkout -b release/${{ env.MILESTONE }} ${GITHUB_SHA}`
  - Line 53: `run: echo -e "${CHANGELOG}" > release-${{ env.MILESTONE }}-changelog.md`
  - Lines 56, 59: `release-${{ env.MILESTONE }}-release-message.md` used inside a multi-line `run:` block.

Locations:

- `.github/workflows/ci.yml:26`
- `.github/workflows/ci.yml:39`
- `.github/workflows/craft-release.yaml:52`
- `.github/workflows/craft-release.yaml:53`
- `.github/workflows/craft-release.yaml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across ci.yml, craft-release.yaml, and set-milestone-on-pr.yaml:

1. unpinned-uses: Pinned all 11 action references to full 40-char SHA digests with original tag/branch as comments. SHAs resolved via lookup_action_sha.

2. missing-permissions: Added top-level permissions blocks to all three files with minimal required permissions (ci.yml: contents:read; craft-release.yaml: contents:write + pull-requests:read; set-milestone-on-pr.yaml: contents:read + pull-requests:write + issues:write).

3. script-injection: In ci.yml, moved step output expressions (${{ steps.previoustag.outputs.tag }}, etc.) into env: blocks as TAG and TIMESTAMP variables. In craft-release.yaml, moved ${{ env.MILESTONE }} references in run: blocks into MILESTONE_VALUE env vars per step. The set-milestone-on-pr.yaml file already had all ${{ }} expressions in env: blocks so no changes were needed there for script injection.

