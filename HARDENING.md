<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the user-controlled input `inputs.sbt-runner-version` is mapped to the `SBT_RUNNER_VERSION` env var and then written directly into `$GITHUB_OUTPUT` via multiple `echo "sbt_toolpath=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"` statements (and similarly for `sbt_cachekey`). No newline sanitization (`printf '%s' ... | tr -d '\n\r'`) is applied before the write. An attacker who controls the `sbt-runner-version` input could inject newlines to smuggle arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

In the 'Setup PATH' step, `SBT_TOOLPATH` is set from `steps.cache-paths.outputs.sbt_toolpath`, which was constructed from the untrusted `inputs.sbt-runner-version`. The script does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to `$GITHUB_PATH` without sanitization (`printf '%s' ... | tr -d '\n\r'`). A newline-containing version input could inject an arbitrary path entry into `$GITHUB_PATH`, enabling PATH hijacking in subsequent steps.

Locations:

- `action.yml:212`
- `action.yml:214`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Added `SBT_RUNNER_VERSION_SAFE=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block and replaced all uses of `$SBT_RUNNER_VERSION` in $GITHUB_OUTPUT writes with the sanitized variable.
2. 'Setup PATH' step: Added `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows equivalent) before writing to $GITHUB_PATH, using the sanitized value in the echo statement.

