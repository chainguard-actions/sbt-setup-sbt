<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the caller-controlled input `inputs.sbt-runner-version` and then written unsanitized into $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A malicious caller could inject newlines into the input to poison subsequent output variables.

Locations:

- `action.yml:23`
- `action.yml:27`
- `action.yml:31`
- `action.yml:35`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is populated from `steps.cache-paths.outputs.sbt_toolpath` (itself derived from the untrusted `inputs.sbt-runner-version`). The script does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH without any sanitization (`echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). A newline-injected version string could poison the PATH.

Locations:

- `action.yml:199`
- `action.yml:201`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Added sanitization of SBT_RUNNER_VERSION using `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` into SAFE_SBT_RUNNER_VERSION before all GITHUB_OUTPUT writes (sbt_toolpath on Windows/macOS/Linux and sbt_cachekey).
2. 'Setup PATH' step: Added sanitization of the path string using `printf '%s' "$PWD/sbt/bin" | tr -d '\n\r'` into safe_path before writing to GITHUB_PATH, for both Windows and non-Windows branches.

