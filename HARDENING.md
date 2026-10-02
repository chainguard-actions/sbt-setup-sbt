<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.23** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is sourced from inputs.sbt-runner-version (attacker-controlled) and written directly into $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-supplied version string containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. Affected lines write: `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:19`
- `action.yml:22`
- `action.yml:25`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the value written to $GITHUB_PATH is derived from $PWD after `cd "$SBT_TOOLPATH"`, where SBT_TOOLPATH comes from steps.cache-paths.outputs.sbt_toolpath — itself built from the attacker-controlled inputs.sbt-runner-version. This creates an indirect injection path: a crafted version string with newlines could cause arbitrary entries to be appended to $GITHUB_PATH without sanitization (no `printf '%s' ... | tr -d '\n\r'` applied before the write).

Locations:

- `action.yml:160`
- `action.yml:162`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (lines 19, 22, 25): Added `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` to sanitize the attacker-controlled `inputs.sbt-runner-version` before writing it into $GITHUB_OUTPUT for sbt_toolpath (Windows and Linux/macOS paths) and sbt_cachekey.

2. 'Setup PATH' step (lines 160, 162): Added `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows equivalent) to sanitize the path value before writing it to $GITHUB_PATH, preventing indirect injection via the version-derived toolpath.

