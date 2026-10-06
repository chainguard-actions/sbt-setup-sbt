<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` (sourced from `inputs.sbt-runner-version`, an attacker-controllable input) is written unsanitized into `$GITHUB_OUTPUT` via multiple `echo` statements (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before these writes. A newline-embedded value in `inputs.sbt-runner-version` could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:23`
- `action.yml:27`
- `action.yml:31`
- `action.yml:35`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var `SBT_TOOLPATH` (sourced from `steps.cache-paths.outputs.sbt_toolpath`, which itself embeds the unsanitized `inputs.sbt-runner-version`) is used as the target of `cd "$SBT_TOOLPATH"`, making `$PWD` attacker-influenced. The resulting `$PWD/sbt/bin` (or `$PWD\\sbt\\bin` on Windows) is then written to `$GITHUB_PATH` via `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` without any sanitization. An attacker-controlled newline in the version input could inject arbitrary paths into `$GITHUB_PATH`.

Locations:

- `action.yml:196`
- `action.yml:198`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and replaced all $SBT_RUNNER_VERSION references in $GITHUB_OUTPUT writes with $SAFE_SBT_RUNNER_VERSION to prevent newline injection.
2. 'Setup PATH' step: Sanitized the path before writing to $GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and the Windows equivalent) to prevent newline injection via the attacker-controlled sbt-runner-version input.

