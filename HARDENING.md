<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the untrusted input `inputs.sbt-runner-version` and then written unsanitized into $GITHUB_OUTPUT multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller supplying a version string containing newlines could inject arbitrary key=value pairs into the GitHub Actions output context.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:41`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is populated from `steps.cache-paths.outputs.sbt_toolpath` (itself derived from the untrusted `inputs.sbt-runner-version`). The script does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization — e.g., `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`. A newline-containing version string could inject additional entries into the PATH, enabling path-hijacking attacks.

Locations:

- `action.yml:207`
- `action.yml:209`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/action.yml:
1. 'Set up cache paths' step: Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the top of the run block to strip newlines from the untrusted `inputs.sbt-runner-version` value before it is embedded in any $GITHUB_OUTPUT writes (sbt_toolpath, sbt_cachekey, etc.).
2. 'Setup PATH' step: Added `SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` to sanitize the toolpath before use, and wrapped the $PWD-derived path in `safe_path=$(printf '%s' "..." | tr -d '\n\r')` before writing to $GITHUB_PATH, preventing newline injection into the PATH context.

