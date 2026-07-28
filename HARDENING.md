<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file .github/workflows/ci.yml references GitHub Actions using mutable tag refs instead of full 40-character commit SHA pins. Specifically: `actions/checkout@v7` and `actions/setup-java@v5`. These can be silently updated by the upstream maintainer, enabling supply-chain attacks. They must be pinned to a full SHA (e.g., `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Locations:

- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:24`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key and no job-level `permissions:` key on the `test` job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal `permissions:` block (e.g., `contents: read`) should be added.

Locations:

- `.github/workflows/ci.yml:1`

### github-env-injection (severity: high)

In action.yml, the 'Set up cache paths' step maps the caller-controlled input `inputs.sbt-runner-version` to the env var `SBT_RUNNER_VERSION`, then writes it unsanitized into `$GITHUB_OUTPUT` multiple times — for example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A newline embedded in the input value could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`) is absent before every such write.

Locations:

- `action.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

1. ci.yml: Pinned actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 # v7 and actions/setup-java@v5 → @03ad4de0992f5dab5e18fcb136590ce7c4a0ac95 # v5. Added top-level `permissions: contents: read` block. 2. action.yml: In the 'Set up cache paths' step, added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, and replaced all occurrences of $SBT_RUNNER_VERSION in GITHUB_OUTPUT writes with $SAFE_SBT_RUNNER_VERSION to prevent newline injection attacks.

