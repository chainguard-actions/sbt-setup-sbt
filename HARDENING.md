<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file .github/workflows/ci.yml references GitHub Actions using mutable tag refs instead of immutable 40-character SHA digests. This exposes the workflow to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v7` and `actions/setup-java@v5`. These should be pinned to their full commit SHAs (e.g., `actions/checkout@<40-char-sha> # v7`).

Locations:

- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:24`

### github-env-injection (severity: high)

In action.yml's 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `inputs.sbt-runner-version` (an attacker-controllable input) and then written directly into `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. A newline embedded in the input value could inject arbitrary key=value pairs into the GitHub Actions output context.

Locations:

- `action.yml:24`
- `action.yml:29`
- `action.yml:34`
- `action.yml:40`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key and no job-level `permissions:` key on the `test` job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents). A minimal explicit `permissions:` block (e.g., `contents: read`) should be added.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

Fixed three findings in two files: (1) Pinned actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 # v7 and actions/setup-java@v5 → @03ad4de0992f5dab5e18fcb136590ce7c4a0ac95 # v5 in .github/workflows/ci.yml. (2) Added top-level `permissions: contents: read` block to .github/workflows/ci.yml. (3) Added sanitization of SBT_RUNNER_VERSION at the start of the 'Set up cache paths' run script in action.yml using `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` to prevent newline injection into $GITHUB_OUTPUT.

