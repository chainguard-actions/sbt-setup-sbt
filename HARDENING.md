<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the untrusted input `${{ inputs.sbt-runner-version }}` and then written directly to $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`, `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). No sanitization (`printf '%s' ... | tr -d '\n\r'`) is applied before any of these writes. An attacker-controlled newline in the input could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:43`

### script-injection (severity: high)

Rule (b) violation: In the 'Download and Install sbt' step, the env var $SBT_RUNNER_VERSION (sourced from the untrusted input `inputs.sbt-runner-version`) is interpolated unquoted inside URL strings passed to curl: `curl -sL "https://github.com/sbt/sbt/releases/download/v$SBT_RUNNER_VERSION/sbt-$SBT_RUNNER_VERSION.zip"`. Although the outer string is double-quoted, the variable itself is not separately quoted and shell metacharacters in the input value (e.g., `$(...)`, backticks) can still be evaluated by the shell before the string is passed to curl. The variable should be validated or the URL constructed with proper quoting.

Locations:

- `action.yml:68`
- `action.yml:70`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed both findings in action.yml:

1. github-env-injection (lines 27, 31, 35, 43): In the 'Set up cache paths' step, added sanitization of SBT_RUNNER_VERSION using `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` to strip newlines/carriage returns, plus a regex validation (`^[0-9]+\.[0-9]+(\.[0-9]+)?$`) to ensure only valid version strings are accepted. All GITHUB_OUTPUT writes now use the sanitized SAFE_SBT_RUNNER_VERSION variable.

2. script-injection (lines 68, 70): In the 'Download and Install sbt' step, added the same sanitization and validation of SBT_RUNNER_VERSION before it is interpolated into curl URL strings and file paths. The validated SAFE_SBT_RUNNER_VERSION is used throughout, preventing shell metacharacter injection (e.g., $(...), backticks) from being evaluated.

