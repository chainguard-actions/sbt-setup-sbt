<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the input `inputs.sbt-runner-version` is mapped to the env var `$SBT_RUNNER_VERSION` and then written directly into `$GITHUB_OUTPUT` multiple times (e.g., `echo "sbt_toolpath=...\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`, `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller-controlled newline in the input value could inject arbitrary key=value pairs into the GitHub output context. This is a case (d) violation: indirect write of inputs via env var without sanitization.

Locations:

- `action.yml:26`

### unpinned-uses (severity: high)

The workflow file ci.yml references two actions using mutable tag-based refs instead of pinned 40-character commit SHAs: `actions/checkout@v6` (line 25) and `actions/setup-java@v5` (line 27). These can be silently updated by the upstream repository, enabling supply-chain attacks.

Locations:

- `.github/workflows/ci.yml:25`
- `.github/workflows/ci.yml:27`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key, and the single `test` job also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the default repository token permissions (which may be read/write), violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed three security findings: (1) github-env-injection in action.yml 'Set up cache paths' step: added sanitization of SBT_RUNNER_VERSION via `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` into SAFE_SBT_RUNNER_VERSION before all GITHUB_OUTPUT writes; (2) unpinned-uses in ci.yml: pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and actions/setup-java@v5 to SHA 03ad4de0992f5dab5e18fcb136590ce7c4a0ac95; (3) missing-permissions in ci.yml: added top-level `permissions: {}` to enforce least privilege.

