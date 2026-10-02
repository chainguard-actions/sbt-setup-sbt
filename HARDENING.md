<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the untrusted input `inputs.sbt-runner-version` (via `env: SBT_RUNNER_VERSION: ${{ inputs.sbt-runner-version }}`). This value is then written directly to $GITHUB_OUTPUT in multiple echo statements (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). An attacker-controlled version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:39`
- `action.yml:45`
- `action.yml:46`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is populated from the untrusted steps output `steps.cache-paths.outputs.sbt_toolpath` (via `env: SBT_TOOLPATH: "${{ steps.cache-paths.outputs.sbt_toolpath }}"`). The script does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` (or `$PWD\\sbt\\bin` on Windows) to $GITHUB_PATH without sanitization. Since $PWD is derived from the untrusted SBT_TOOLPATH value, a newline-containing path could inject arbitrary entries into $GITHUB_PATH, enabling PATH hijacking for subsequent steps.

Locations:

- `action.yml:211`
- `action.yml:213`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Sanitized SBT_RUNNER_VERSION (from untrusted input) by computing SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r') at the start of the script, then using SAFE_SBT_RUNNER_VERSION in all GITHUB_OUTPUT writes (sbt_toolpath and sbt_cachekey lines).
2. 'Setup PATH' step: Sanitized the path before writing to GITHUB_PATH by capturing it in safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r') (and the Windows equivalent), then echoing $safe_path to GITHUB_PATH instead of the raw $PWD-derived value.

