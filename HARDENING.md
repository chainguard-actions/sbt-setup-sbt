<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version via the env: block) is written to $GITHUB_OUTPUT on multiple lines without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). An attacker-controlled version string containing newlines could inject arbitrary key=value pairs into the GitHub output context. Affected lines write: echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT" (Windows, macOS, and Linux branches) and echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT".

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the value written to $GITHUB_PATH is derived from $SBT_TOOLPATH (set from steps.cache-paths.outputs.sbt_toolpath, which was itself constructed from the user-controlled inputs.sbt-runner-version). The script does `cd "$SBT_TOOLPATH"` then `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` without any sanitization (no printf '%s' ... | tr -d '\n\r'). A newline-containing version input could inject arbitrary entries into $GITHUB_PATH.

Locations:

- `action.yml:27`
- `action.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Added sanitization of SBT_RUNNER_VERSION via `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script, then replaced all $SBT_RUNNER_VERSION references in GITHUB_OUTPUT writes with $SAFE_SBT_RUNNER_VERSION (Windows, macOS, Linux toolpath lines and the sbt_cachekey line).
2. 'Setup PATH' step: Added sanitization of the path before writing to $GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows equivalent), then writing $safe_path to $GITHUB_PATH instead of the raw value.

