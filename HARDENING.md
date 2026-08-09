<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file ci.yml uses mutable tag references instead of pinned 40-character SHA commit hashes. `actions/checkout@v7` and `actions/setup-java@v5` are both tag refs that can be silently redirected to different code, enabling supply-chain attacks.

Locations:

- `.github/workflows/ci.yml:21`
- `.github/workflows/ci.yml:23`

### permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the default (potentially write-all) token permissions, granting broader access than necessary.

Locations:

- `.github/workflows/ci.yml:1`

### github-env-injection (severity: high)

Two steps in action.yml write values derived from untrusted inputs to special GitHub environment files without the required sanitization (`printf '%s' ... | tr -d '\n\r'`).

1. **"Set up cache paths" step**: `SBT_RUNNER_VERSION` is set from `inputs.sbt-runner-version` (caller-controlled) via the `env:` block, then written directly into `$GITHUB_OUTPUT` in multiple lines such as `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. A newline embedded in the input value would inject arbitrary key=value pairs into GITHUB_OUTPUT.

2. **"Setup PATH" step**: `SBT_TOOLPATH` is set from `steps.cache-paths.outputs.sbt_toolpath` (which itself contains the unsanitized input-derived version string) via the `env:` block, then after `cd "$SBT_TOOLPATH"`, the resulting `$PWD/sbt/bin` path is written to `$GITHUB_PATH` without sanitization. A newline in the toolpath value would inject arbitrary entries into GITHUB_PATH.

Locations:

- `action.yml:21`
- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:39`
- `action.yml:43`
- `action.yml:47`
- `action.yml:51`
- `action.yml:55`
- `action.yml:57`
- `action.yml:155`
- `action.yml:160`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, github-env-injection

**Notes:**

Fixed all three findings: (1) Pinned actions/checkout@v7 to full SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 and actions/setup-java@v5 to b6effb05e454b25005698d916606bdc6ffcbf961 in ci.yml. (2) Added top-level 'permissions: {}' and job-level 'permissions: contents: read' to ci.yml. (3) Fixed github-env-injection in action.yml: sanitized SBT_RUNNER_VERSION with 'printf | tr -d newlines' before writing to GITHUB_OUTPUT in the 'Set up cache paths' step, and sanitized SBT_TOOLPATH before using it in the 'Setup PATH' step that writes to GITHUB_PATH.

