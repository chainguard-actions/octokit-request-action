<!-- markdownlint-disable -->

# Hardening Report: octokit--request-action/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **octokit--request-action/v3.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or version strings instead of full 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the tag is moved.

Failing references:
- add_to_octokit_project.yml: `actions/add-to-project@v1.0.2`
- codeql-analysis.yml: `actions/checkout@v6`, `github/codeql-action/init@v4`, `github/codeql-action/autobuild@v4`, `github/codeql-action/analyze@v4`
- immediate-response.yml: `peter-evans/create-or-update-comment@v5`
- publish-immutable-actions.yml: `actions/checkout@v6`, `actions/publish-immutable-action@0.0.4`
- release.yml: `actions/create-github-app-token@v3`, `actions/checkout@v6`, `actions/setup-node@v6`
- test.yml: `actions/checkout@v6`, `actions/setup-node@v6` (multiple occurrences)

Locations:

- `.github/workflows/add_to_octokit_project.yml:14`
- `.github/workflows/codeql-analysis.yml:37`
- `.github/workflows/codeql-analysis.yml:41`
- `.github/workflows/codeql-analysis.yml:52`
- `.github/workflows/codeql-analysis.yml:63`
- `.github/workflows/immediate-response.yml:22`
- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:17`
- `.github/workflows/test.yml:10`
- `.github/workflows/test.yml:11`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:23`
- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:58`

### missing-permissions (severity: medium)

The workflow file `add_to_octokit_project.yml` has no top-level `permissions:` key and its only job (`add-to-project`) also has no job-level `permissions:` key. Without explicit permissions, the job inherits the default (potentially broad) repository token permissions.

Locations:

- `.github/workflows/add_to_octokit_project.yml:1`

### script-injection (severity: high)

Rule (a) violation: The `run:` block in the 'Determine issue or PR number' step directly interpolates the GitHub Actions expression `${{ github.event.issue.number || github.event.pull_request.number }}` inside a shell command string. This value is attacker-controlled (it comes from issue/PR events, including `pull_request_target`) and is substituted into the shell command before the shell ever sees it, enabling command injection. Offending line: `run: echo "NUMBER=${{ github.event.issue.number || github.event.pull_request.number }}" >> "$GITHUB_OUTPUT"`

Locations:

- `.github/workflows/immediate-response.yml:21`

### github-env-injection (severity: high)

The same `run:` step writes the attacker-controlled expression `${{ github.event.issue.number || github.event.pull_request.number }}` directly to `$GITHUB_OUTPUT` without first applying the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). This allows an attacker to inject newlines into the output file and poison subsequent steps that read the `NUMBER` output. The workflow is triggered by `pull_request_target`, making this reachable from untrusted forks. Offending line: `echo "NUMBER=${{ github.event.issue.number || github.event.pull_request.number }}" >> "$GITHUB_OUTPUT"`

Locations:

- `.github/workflows/immediate-response.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all findings across 6 workflow files:

1. **unpinned-uses**: Pinned all 17 action references to full SHA digests:
   - actions/add-to-project@v1.0.2 → @244f685bbc3b7adfa8466e08b698b5577571133e
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (5 occurrences)
   - github/codeql-action/init@v4 → @db488ddef3bf6cb639b32c2e9a7c0a7ea8271d28
   - github/codeql-action/autobuild@v4 → @db488ddef3bf6cb639b32c2e9a7c0a7ea8271d28
   - github/codeql-action/analyze@v4 → @db488ddef3bf6cb639b32c2e9a7c0a7ea8271d28
   - peter-evans/create-or-update-comment@v5 → @e8674b075228eee787fea43ef493e45ece1004c9
   - actions/publish-immutable-action@0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978 (resolved as v0.0.4)
   - actions/create-github-app-token@v3 → @bcd2ba49218906704ab6c1aa796996da409d3eb1
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 (3 occurrences)

2. **missing-permissions**: Added `permissions: {}` top-level block to add_to_octokit_project.yml.

3. **script-injection**: In immediate-response.yml, moved `${{ github.event.issue.number || github.event.pull_request.number }}` into an `env:` block as `ISSUE_NUMBER` and referenced it as `$ISSUE_NUMBER` in the shell script.

4. **github-env-injection**: In immediate-response.yml, sanitized the value with `printf '%s' "$ISSUE_NUMBER" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in hardened/action/.github/workflows/test.yml:
1. Line 41: Moved `${{ fromJson(steps.get_latest_release.outputs.data).tag_name }}` into an `env:` block as `TAG_NAME`, then referenced it as `"$TAG_NAME"` in the shell command.
2. Line 68: Moved `${{ steps.get_release.outputs.status }}` into an `env:` block as `RELEASE_STATUS`, then referenced it as `"$RELEASE_STATUS"` in the shell command.
Both fixes follow the pattern of isolating GitHub expression interpolation from shell execution to prevent script injection attacks.

