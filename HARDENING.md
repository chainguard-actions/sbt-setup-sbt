<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.2** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the user-controlled input `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION` and then writes it directly to `$GITHUB_OUTPUT` multiple times without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A calling workflow could supply a version string containing embedded newlines to inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the run script to sanitize the user-controlled `inputs.sbt-runner-version` value (mapped to env var SBT_RUNNER_VERSION) before it is written to $GITHUB_OUTPUT multiple times. This prevents an attacker from injecting arbitrary key=value pairs into GITHUB_OUTPUT via embedded newlines in the version string.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml (lines 193/195). The step was writing $PWD (derived from the untrusted $SBT_TOOLPATH step output) directly to $GITHUB_PATH. The fix sanitizes the path values using `printf '%s' "$PWD/..." | tr -d '\n\r'` before writing to $GITHUB_PATH, preventing newline injection attacks that could poison $GITHUB_PATH with arbitrary additional entries.

