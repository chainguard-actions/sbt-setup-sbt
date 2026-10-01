<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.23** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version) is written directly into $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` applied before the write). An attacker controlling the sbt-runner-version input can inject newline characters to poison GITHUB_OUTPUT with arbitrary key=value pairs. Affected lines: `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows and Linux branches) and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:19`
- `action.yml:22`
- `action.yml:25`

### github-env-injection (severity: high)

In the 'Setup PATH' step, SBT_TOOLPATH (sourced from steps.cache-paths.outputs.sbt_toolpath, which was derived from the untrusted input inputs.sbt-runner-version) is used via `cd "$SBT_TOOLPATH"` and then `$PWD/sbt/bin` is written to $GITHUB_PATH without sanitization. An attacker controlling sbt-runner-version can inject newlines into the path written to GITHUB_PATH, potentially prepending attacker-controlled directories to the runner's PATH.

Locations:

- `action.yml:210`
- `action.yml:212`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (lines 19, 22, 25): Added `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, then replaced all three uses of `$SBT_RUNNER_VERSION` in the GITHUB_OUTPUT echo commands with `$safe_version`. This prevents newline injection via the sbt-runner-version input.

2. 'Setup PATH' step (lines 210, 212): Added sanitization of the path before writing to GITHUB_PATH. Both Windows and Linux branches now compute `safe_path=$(printf '%s' "$PWD/..." | tr -d '\n\r')` and write `$safe_path` to `$GITHUB_PATH` instead of the raw PWD-derived value. This prevents newline injection via the sbt_toolpath output (which was derived from the untrusted sbt-runner-version input).

