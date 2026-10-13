<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.12** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from inputs.sbt-runner-version (an untrusted caller-controlled input) via env:, then writes it directly into $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' ... | tr -d '\n\r'). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A malicious caller could inject newlines into the version string to poison subsequent output variables or environment entries read from $GITHUB_OUTPUT.

Locations:

- `action.yml:28`

### github-env-injection (severity: high)

The 'Setup PATH' step sets SBT_TOOLPATH from steps.cache-paths.outputs.sbt_toolpath (which was derived from the untrusted inputs.sbt-runner-version) via env:, then does `cd "$SBT_TOOLPATH"` and writes `$PWD/sbt/bin` (or `$PWD\\sbt\\bin` on Windows) to $GITHUB_PATH without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Because $PWD is rooted in a path containing the caller-controlled version string, a newline-containing version value could inject additional entries into $GITHUB_PATH.

Locations:

- `action.yml:196`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (line 28): Added `SAFE_SBT_RUNNER_VERSION="$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')"` to sanitize the untrusted caller-controlled input before using it in all `echo ... >> "$GITHUB_OUTPUT"` statements that include the version string (sbt_toolpath and sbt_cachekey outputs).

2. 'Setup PATH' step (line 196): Added sanitization of the path before writing to `$GITHUB_PATH`. The path (which is rooted in the caller-controlled version string) is now sanitized with `printf '%s' "$PWD/sbt/bin" | tr -d '\n\r'` (and the Windows equivalent with backslashes) before being echoed to `$GITHUB_PATH`.

