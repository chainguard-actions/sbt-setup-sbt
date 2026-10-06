<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the untrusted input `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION` and then writes it directly to `$GITHUB_OUTPUT` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Multiple writes are affected, e.g.:
  `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`
  `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`
A caller supplying a version string containing newlines could inject arbitrary key=value pairs into the runner's output environment (case d violation).

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

The 'Setup PATH' step maps `steps.cache-paths.outputs.sbt_toolpath` (which was computed from the unsanitized `inputs.sbt-runner-version`) into the env var `SBT_TOOLPATH`, then does `cd "$SBT_TOOLPATH"` and writes `$PWD/sbt/bin` to `$GITHUB_PATH` without sanitization:
  `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`
Because `sbt_toolpath` is derived from user-controlled input and is never sanitized before being forwarded to `$GITHUB_PATH`, a caller can inject arbitrary entries into the runner's PATH (case c/e violation).

Locations:

- `action.yml:196`
- `action.yml:198`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Sanitized SBT_RUNNER_VERSION by adding `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, then replaced all uses of $SBT_RUNNER_VERSION in GITHUB_OUTPUT writes with $SAFE_SBT_RUNNER_VERSION. This prevents newline injection into sbt_toolpath and sbt_cachekey outputs.
2. 'Setup PATH' step: Sanitized SBT_TOOLPATH by adding `SAFE_SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` at the start of the run block, then used $SAFE_SBT_TOOLPATH for the cd command. The $PWD/sbt/bin value written to GITHUB_PATH is then derived from the shell's own $PWD (set after cd to the sanitized path), preventing injection.

