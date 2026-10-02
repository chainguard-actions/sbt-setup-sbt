<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.23** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the untrusted input `inputs.sbt-runner-version` and then written directly into $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-controlled version string containing newlines could inject arbitrary key=value pairs into the output context. Affected lines write: `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (both Windows and Linux branches) and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:19`
- `action.yml:22`
- `action.yml:25`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is populated from `steps.cache-paths.outputs.sbt_toolpath` (which is itself derived from the untrusted `inputs.sbt-runner-version`). The script does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization. A newline-containing version input could cause SBT_TOOLPATH to resolve to a path that injects additional entries into $GITHUB_PATH.

Locations:

- `action.yml:246`
- `action.yml:248`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Added `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and replaced all three GITHUB_OUTPUT writes to use `$safe_version` instead of `$SBT_RUNNER_VERSION`, preventing newline injection from the untrusted `inputs.sbt-runner-version`.
2. 'Setup PATH' step: Added `safe_toolpath=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` and replaced `cd "$SBT_TOOLPATH"` with `cd "$safe_toolpath"`, so the `$PWD`-derived path written to GITHUB_PATH cannot contain injected newlines.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In the 'Download and Install sbt' step, added sanitization of the attacker-controlled `SBT_RUNNER_VERSION` input by computing `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, then replaced all three occurrences of `$SBT_RUNNER_VERSION` in the curl URLs and unzip command with `$safe_version`. This prevents command substitution injection (e.g., a version string like `$(malicious_command)`) from being evaluated in the shell. The fix is consistent with the sanitization pattern already used in the 'Set up cache paths' step.

