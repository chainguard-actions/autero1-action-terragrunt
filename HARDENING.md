<!-- markdownlint-disable -->

# Hardening Report: autero1--action-terragrunt/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **autero1--action-terragrunt/v3.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of user-controlled values inside run: blocks. In update-main-version.yml, `${{ github.event.inputs.major_version }}` and `${{ github.event.inputs.target }}` are interpolated directly into shell commands (`git tag -f` and `git push origin`), allowing an attacker with workflow_dispatch access to inject arbitrary shell commands. In test-action.yml and test.yml, `${{ contains(github.event.head_commit.message, '[skip ci]') }}` is interpolated directly into an `echo` run: command — any ${{ ... }} expression inside a run: block is a script-injection risk regardless of context.

Locations:

- `.github/workflows/update-main-version.yml:28`
- `.github/workflows/update-main-version.yml:30`
- `.github/workflows/test-action.yml:22`
- `.github/workflows/test.yml:23`

### github-env-injection (severity: high)

In release-please.yml, the variable `${DIFF}` is populated from `git log` output (which can contain attacker-controlled commit messages crafted by anyone who can push a commit) and is written directly to $GITHUB_ENV via `echo "${DIFF}" >> $GITHUB_ENV` without the required sanitization step (`printf '%s' "$DIFF" | tr -d '\n\r'`). A commit message containing a newline followed by `KEY=value` would inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/release-please.yml:41`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tag refs instead of immutable 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or the upstream repository is compromised. Failing references include: release-please.yml: actions/checkout@v4, actions/setup-node@v4, peter-evans/create-pull-request@v6, google-github-actions/release-please-action@v4; stale.yml: actions/stale@v9; test-action release-version.yml: autero1/action-terragrunt@v3 (×2); test-action.yml: actions/checkout@v4, actions/setup-node@v4; test.yml: actions/checkout@v4, actions/setup-node@v4; update-main-version.yml: actions/checkout@v4.

Locations:

- `.github/workflows/release-please.yml:13`
- `.github/workflows/release-please.yml:14`
- `.github/workflows/release-please.yml:44`
- `.github/workflows/release-please.yml:56`
- `.github/workflows/stale.yml:10`
- `.github/workflows/test-action release-version.yml:9`
- `.github/workflows/test-action release-version.yml:13`
- `.github/workflows/test-action.yml:33`
- `.github/workflows/test-action.yml:36`
- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:37`
- `.github/workflows/update-main-version.yml:19`

### missing-permissions (severity: medium)

None of the 6 workflow files under .github/workflows/ declare a top-level `permissions:` block, and no individual job within any of these files declares a `permissions:` block either. Without explicit permissions, workflows run with the default token permissions (which may be read/write depending on repository settings), violating the principle of least privilege. All six files are affected: release-please.yml, stale.yml, test-action release-version.yml, test-action.yml, test.yml, and update-main-version.yml.

Locations:

- `.github/workflows/release-please.yml:1`
- `.github/workflows/stale.yml:1`
- `.github/workflows/test-action release-version.yml:1`
- `.github/workflows/test-action.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-main-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 finding types across 6 workflow files:

1. script-injection: Moved ${{ github.event.inputs.major_version }} and ${{ github.event.inputs.target }} to env: blocks in update-main-version.yml. Moved ${{ contains(...) }} expressions to env: blocks in test-action.yml and test.yml skipci steps.

2. github-env-injection: In release-please.yml, sanitized CURRENT_HASH and LAST_BUILD_HASH with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_ENV. For the multiline DIFF value (which uses heredoc syntax), used `printf '%s' | tr -d '\r'` to strip carriage returns while preserving legitimate newlines within the heredoc block.

3. unpinned-uses: Pinned all 12 action references to full 40-char SHAs: actions/checkout@v4→11d5960a, actions/setup-node@v4→49933ea5, peter-evans/create-pull-request@v6→c5a7806, google-github-actions/release-please-action@v4→e4dc86ba, actions/stale@v9→5bef64f1, autero1/action-terragrunt@v3→aefb0a43 (×2).

4. missing-permissions: Added top-level permissions blocks to all 6 workflow files with least-privilege permissions appropriate to each workflow's function.

