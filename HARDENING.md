<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.9** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In action.yml, the 'Set up cache paths' step writes the value of $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version via the env: block) to $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). An attacker-controlled version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. Affected writes include: echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT" (Windows, macOS, Linux branches) and echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT".

Locations:

- `action.yml:14`

### unpinned-uses (severity: high)

The workflow file .github/workflows/ci.yml references two actions using mutable tag refs instead of immutable 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if those tags are moved: (1) uses: actions/checkout@v7 — should be pinned to a full SHA; (2) uses: actions/setup-java@v6 — should be pinned to a full SHA.

Locations:

- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:24`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level permissions: key and the single job 'test' also has no job-level permissions: key. Without explicit permissions, the workflow inherits the default repository token permissions (which may be write-all depending on repository settings), violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses, missing-permissions

**Notes:**

1. github-env-injection (action.yml): Added sanitization of the attacker-controlled SBT_RUNNER_VERSION input at the start of the 'Set up cache paths' run block using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All GITHUB_OUTPUT writes that included the version string now use $SAFE_SBT_RUNNER_VERSION instead. 2. unpinned-uses (.github/workflows/ci.yml): Pinned actions/checkout@v7 to SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 and actions/setup-java@v6 to SHA dd06d9cba3e5552c54d9f8ea23572deb30010f7c. 3. missing-permissions (.github/workflows/ci.yml): Added top-level `permissions: {}` to enforce least privilege.

