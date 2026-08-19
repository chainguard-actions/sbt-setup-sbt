<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the env var $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version via the env: block) is written directly into $GITHUB_OUTPUT in multiple echo statements (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A caller can supply a version string containing newline characters to inject arbitrary key=value pairs into GITHUB_OUTPUT. Additionally, in the 'Setup PATH' step, $SBT_TOOLPATH (sourced from steps.cache-paths.outputs.sbt_toolpath, itself derived from the unsanitized input) is written to $GITHUB_PATH without sanitization.

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:39`
- `action.yml:44`
- `action.yml:46`
- `action.yml:47`
- `action.yml:196`
- `action.yml:198`

### unpinned-uses (severity: high)

The workflow file ci.yml references two actions using mutable version tags instead of pinned 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if those tags are moved: `actions/checkout@v6` and `actions/setup-java@v5`.

Locations:

- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:24`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key and the single job 'test' also has no `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions (which may include write access to contents and other scopes), violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed three findings: (1) github-env-injection in action.yml - sanitized SBT_RUNNER_VERSION before writing to GITHUB_OUTPUT using `printf '%s' | tr -d '\n\r'` in the 'Set up cache paths' step, and sanitized SBT_TOOLPATH before using it in the 'Setup PATH' step; (2) unpinned-uses in ci.yml - pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and actions/setup-java@v5 to SHA 03ad4de0992f5dab5e18fcb136590ce7c4a0ac95; (3) missing-permissions in ci.yml - added top-level `permissions: {}` to enforce least privilege.

