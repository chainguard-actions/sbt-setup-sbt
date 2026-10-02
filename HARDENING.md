<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.11** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the untrusted input `inputs.sbt-runner-version` and then written directly into $GITHUB_OUTPUT in multiple echo statements (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). None of these writes are preceded by the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller supplying a version string containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`
- `action.yml:43`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is set from `steps.cache-paths.outputs.sbt_toolpath` (which was constructed from the untrusted `inputs.sbt-runner-version`). The step then does `cd "$SBT_TOOLPATH"` and writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization (e.g., `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). A newline-containing version string could inject additional entries into GITHUB_PATH, allowing PATH hijacking for subsequent steps.

Locations:

- `action.yml:212`
- `action.yml:214`

### script-injection (severity: high)

Sub-rule (b): In the 'Download and Install sbt' step, the env var SBT_RUNNER_VERSION (sourced from `inputs.sbt-runner-version`) is expanded unquoted — or inside a double-quoted string where command substitution is still active — in curl URL arguments: `curl -sL "https://github.com/sbt/sbt/releases/download/v$SBT_RUNNER_VERSION/sbt-$SBT_RUNNER_VERSION.zip"`. Within bash double-quotes, `$(...)` and backtick substitutions are still evaluated. A caller supplying a version string such as `$(malicious_command)` would cause arbitrary command execution on the runner.

Locations:

- `action.yml:107`
- `action.yml:108`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Three security fixes applied to hardened/action/action.yml:

1. **github-env-injection (Set up cache paths)**: Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the 'Set up cache paths' run block. All GITHUB_OUTPUT writes that included the version string now use `$SAFE_SBT_RUNNER_VERSION` instead of `$SBT_RUNNER_VERSION`, preventing newline injection into GITHUB_OUTPUT.

2. **github-env-injection (Setup PATH)**: In the 'Setup PATH' step, the path is now sanitized with `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and the Windows equivalent) before being written to $GITHUB_PATH, preventing newline injection into GITHUB_PATH.

3. **script-injection (Download and Install sbt)**: Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the 'Download and Install sbt' run block. All curl URL arguments and file path references now use `${SAFE_SBT_RUNNER_VERSION}` instead of `$SBT_RUNNER_VERSION`, preventing command substitution attacks from a malicious version string containing `$(...)` or backtick expressions.

