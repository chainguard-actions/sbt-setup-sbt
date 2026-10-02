<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.7** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from inputs.sbt-runner-version in its env: block, then writes it unsanitized into $GITHUB_OUTPUT multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. A caller-controlled version string containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent before every such write.

Locations:

- `action.yml:14`

### github-env-injection (severity: high)

The 'Setup PATH' step writes `$PWD/sbt/bin` to $GITHUB_PATH after `cd "$SBT_TOOLPATH"`, where SBT_TOOLPATH is sourced from steps.cache-paths.outputs.sbt_toolpath — a value constructed from the caller-controlled inputs.sbt-runner-version without sanitization. A newline-containing version input could inject an arbitrary extra path entry into GITHUB_PATH. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write.

Locations:

- `action.yml:285`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step (line 14): Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script to sanitize the caller-controlled inputs.sbt-runner-version before it is embedded in any GITHUB_OUTPUT writes (sbt_toolpath, sbt_cachekey, etc.).
2. 'Setup PATH' step (line 285): Added `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows equivalent with backslashes) before writing to GITHUB_PATH, preventing newline injection via the SBT_TOOLPATH value derived from the unsanitized version string.

