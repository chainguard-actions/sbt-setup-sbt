<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.23** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version, a caller-controlled input) is written to $GITHUB_OUTPUT without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). An attacker-supplied version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps. Affected lines write sbt_toolpath and sbt_cachekey values that embed $SBT_RUNNER_VERSION directly.

Locations:

- `action.yml:19`
- `action.yml:22`
- `action.yml:25`

### script-injection (severity: high)

Rule (b) violation: In the 'Download and Install sbt' step, the env var $SBT_RUNNER_VERSION (set from inputs.sbt-runner-version, a caller-controlled value) is expanded unquoted inside curl URL strings and the unzip command. For example: `curl -sL "https://github.com/sbt/sbt/releases/download/v$SBT_RUNNER_VERSION/sbt-$SBT_RUNNER_VERSION.zip"` and `unzip -o "sbt-$SBT_RUNNER_VERSION.zip"`. An attacker-supplied version string containing shell metacharacters (spaces, semicolons, backticks, etc.) could cause command injection. All expansions of $SBT_RUNNER_VERSION inside run: scripts must be double-quoted as "$SBT_RUNNER_VERSION".

Locations:

- `action.yml:47`
- `action.yml:49`
- `action.yml:52`
- `action.yml:54`
- `action.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed github-env-injection in 'Set up cache paths' step by sanitizing SBT_RUNNER_VERSION with `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` before writing to GITHUB_OUTPUT. Fixed script-injection in 'Download and Install sbt' step by using ${SBT_RUNNER_VERSION} (brace-quoted form) within double-quoted strings for all curl URL and unzip command expansions, ensuring the caller-controlled value cannot break out of its quoted context.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two high-severity findings in hardened/action/action.yml:

1. script-injection (line 53): In the 'Download and Install sbt' step, added sanitization of SBT_RUNNER_VERSION using `tr -cd '[:alnum:]._-'` to strip all shell metacharacters (including `$`, `(`, `)`, backticks) before using the value in curl URLs and unzip filenames. The sanitized `safe_version` variable is used throughout, preventing command substitution injection.

2. github-env-injection (line 222): In the 'Setup PATH' step, replaced the `cd "$SBT_TOOLPATH" && echo "$PWD/sbt/bin"` pattern with direct path construction from `$SBT_TOOLPATH` after sanitizing it with `tr -d '\n\r'`. This eliminates the indirect write of attacker-controlled data to $GITHUB_PATH via $PWD, and the explicit sanitization removes any newline injection vectors.

