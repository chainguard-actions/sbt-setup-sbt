<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In action.yml, the 'Set up cache paths' step sets SBT_RUNNER_VERSION from the untrusted input `inputs.sbt-runner-version` via env: and then writes it directly to $GITHUB_OUTPUT multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A newline injected into the input can poison subsequent output variables. Additionally, the 'Setup PATH' step writes a value derived from SBT_TOOLPATH (itself sourced from the unsanitized step output containing the user-controlled version) to $GITHUB_PATH without sanitization.

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:39`
- `action.yml:44`

### unpinned-uses (severity: high)

The workflow file ci.yml references two actions by mutable tag rather than a full 40-character commit SHA, making the workflow vulnerable to supply-chain attacks if the tag is moved: `actions/checkout@v7` and `actions/setup-java@v5`. These should be pinned to their exact commit SHAs (e.g., `actions/checkout@<40-char-sha> # v7`).

Locations:

- `.github/workflows/ci.yml:23`
- `.github/workflows/ci.yml:25`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key and the single job ('test') also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, packages, etc.). A minimal permissions block (e.g., `permissions: read-all` or specific scopes) should be added.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed three security findings: (1) github-env-injection in action.yml - sanitized user-controlled `sbt-runner-version` input with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT in the 'Set up cache paths' step, and similarly sanitized SBT_TOOLPATH before writing to $GITHUB_PATH in the 'Setup PATH' step; (2) unpinned-uses in ci.yml - pinned actions/checkout@v7 to full SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 and actions/setup-java@v5 to full SHA 03ad4de0992f5dab5e18fcb136590ce7c4a0ac95; (3) missing-permissions in ci.yml - added top-level `permissions: contents: read` block to restrict the GITHUB_TOKEN to the minimum required access.

