<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from `inputs.sbt-runner-version` in its env: block and then writes it unsanitized to $GITHUB_OUTPUT in multiple echo statements (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). A caller-supplied version string containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent before every write.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`
- `action.yml:43`

### github-env-injection (severity: high)

The 'Setup PATH' step writes `$PWD` to $GITHUB_PATH without sanitization (`echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). $PWD is computed by `cd "$SBT_TOOLPATH"` where $SBT_TOOLPATH is sourced from step outputs that were themselves built from the caller-controlled `inputs.sbt-runner-version`. A version string containing path-traversal or newline characters could inject additional entries into GITHUB_PATH. The required sanitization step is absent before the write.

Locations:

- `action.yml:242`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/action.yml:

1. 'Set up cache paths' step (lines 27-43): Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block to sanitize the caller-controlled version string before it is embedded in any $GITHUB_OUTPUT writes.

2. 'Setup PATH' step (line 242): Replaced direct `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` with a sanitized form: `safe_pwd=$(printf '%s' "$PWD" | tr -d '\n\r')` followed by `echo "${safe_pwd}/sbt/bin" >> "$GITHUB_PATH"` (and equivalent for Windows). This prevents newline injection into GITHUB_PATH via a crafted sbt-runner-version input.

