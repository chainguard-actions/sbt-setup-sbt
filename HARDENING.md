<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.8** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file .github/workflows/ci.yml uses tag-based (non-SHA-pinned) action references, which are vulnerable to supply-chain attacks if the tag is moved. Failing references:
- `uses: actions/checkout@v7` (line 22) — should be pinned to a full 40-character commit SHA
- `uses: actions/setup-java@v5` (line 24) — should be pinned to a full 40-character commit SHA

Locations:

- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:24`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key, and the single `test` job also has no job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents). A minimal permissions block such as `permissions: read-all` or specific scopes should be added.

Locations:

- `.github/workflows/ci.yml:1`

### github-env-injection (severity: high)

In action.yml, the 'Set up cache paths' step sets `SBT_RUNNER_VERSION` from the caller-controlled input `inputs.sbt-runner-version` via an `env:` block, then writes that value directly into `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). For example:
  `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`
A malicious caller could supply a newline-containing version string to inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. The same unsanitized value also appears in the sbt_cachekey output line.

Locations:

- `action.yml:26`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

1. Pinned actions/checkout@v7 → SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 and actions/setup-java@v5 → SHA b6effb05e454b25005698d916606bdc6ffcbf961 in .github/workflows/ci.yml. 2. Added top-level `permissions: contents: read` to ci.yml to restrict the GITHUB_TOKEN to the minimum needed. 3. In action.yml 'Set up cache paths' step, added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the top of the run block to strip newlines from the caller-controlled input before it is written to $GITHUB_OUTPUT, preventing newline injection attacks.

