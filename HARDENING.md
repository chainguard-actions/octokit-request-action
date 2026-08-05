<!-- markdownlint-disable -->

# Hardening Report: octokit--request-action/v2.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **octokit--request-action/v2.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference external actions using mutable tags or version strings instead of full 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if a tag is moved or an action is compromised. Affected references include: `actions/add-to-project@v1.0.2` (add_to_octokit_project.yml), `actions/checkout@v4`, `actions/setup-node@v4`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3` (codeql-analysis.yml), `peter-evans/create-or-update-comment@v4` (immediate-response.yml), `actions/checkout@v4`, `actions/publish-immutable-action@0.0.3` (publish-immutable-actions.yml), `actions/checkout@v4`, `actions/setup-node@v4` (release.yml), `actions/checkout@v4`, `actions/setup-node@v4` (test.yml).

Locations:

- `.github/workflows/add_to_octokit_project.yml:14`
- `.github/workflows/codeql-analysis.yml:35`
- `.github/workflows/codeql-analysis.yml:40`
- `.github/workflows/codeql-analysis.yml:47`
- `.github/workflows/codeql-analysis.yml:53`
- `.github/workflows/immediate-response.yml:23`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:19`
- `.github/workflows/test.yml:9`
- `.github/workflows/test.yml:10`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:23`

### missing-permissions (severity: medium)

Two workflow files lack a top-level `permissions:` block and have jobs without job-level `permissions:` blocks, meaning those jobs run with the default (potentially broad) token permissions. `add_to_octokit_project.yml` has no `permissions:` key at any level. `test.yml` has no top-level `permissions:` and the `lint` and `issues` jobs each lack a job-level `permissions:` block (only the `readmeExample` job defines one).

Locations:

- `.github/workflows/add_to_octokit_project.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands via controlled context values.

1. `immediate-response.yml` line 21: `run: echo "NUMBER=${{ github.event.issue.number || github.event.pull_request.number }}" >> "$GITHUB_OUTPUT"` — the issue/PR number is attacker-controlled via a `pull_request_target` trigger and is interpolated directly into the shell command.

2. `test.yml` lines 35–36: `run: >-` / `echo latest release: '${{ fromJson(steps.get_latest_release.outputs.data).tag_name }}'` — a step output expression is interpolated directly into a shell echo command.

3. `test.yml` lines 72–73: `run: >-` / `echo Release could not be found. Request failed with status ${{ steps.get_release.outputs.status }}` — a step output expression is interpolated directly into a shell echo command.

Locations:

- `.github/workflows/immediate-response.yml:21`
- `.github/workflows/test.yml:35`
- `.github/workflows/test.yml:72`

### github-env-injection (severity: high)

In `immediate-response.yml` line 21, the value `${{ github.event.issue.number || github.event.pull_request.number }}` — sourced from attacker-controlled event data via a `pull_request_target` trigger — is written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious actor could craft an issue or PR number containing newline characters to inject arbitrary key-value pairs into the GitHub output environment. The offending line is: `run: echo "NUMBER=${{ github.event.issue.number || github.event.pull_request.number }}" >> "$GITHUB_OUTPUT"`

Locations:

- `.github/workflows/immediate-response.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all findings across 6 workflow files:

1. **unpinned-uses**: Pinned all action references to full SHA hashes:
   - actions/add-to-project@v1.0.2 → @244f685bbc3b7adfa8466e08b698b5577571133e
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - github/codeql-action/init@v3 → @c4dd10e44af883a891fe31ced449bcb4a6728b9b
   - github/codeql-action/autobuild@v3 → @c4dd10e44af883a891fe31ced449bcb4a6728b9b
   - github/codeql-action/analyze@v3 → @c4dd10e44af883a891fe31ced449bcb4a6728b9b
   - peter-evans/create-or-update-comment@v4 → @71345be0265236311c031f5c7866368bd1eff043
   - actions/publish-immutable-action@0.0.3 → @4b1aa5c1cde5fedc80d52746c9546cb5560e5f53 (resolved as v0.0.3)

2. **missing-permissions**: Added `permissions: {}` at top level of add_to_octokit_project.yml and test.yml; added `permissions: {}` at job level for `lint` and `issues` jobs in test.yml.

3. **script-injection**: Moved all `${{ }}` expressions in `run:` steps to `env:` blocks and referenced them as plain environment variables in the shell scripts.

4. **github-env-injection**: In immediate-response.yml, sanitized the issue/PR number with `printf '%s' "$ISSUE_NUMBER" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.

