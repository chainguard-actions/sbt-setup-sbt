<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the user-controlled input `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION` and then writes it to `$GITHUB_OUTPUT` in multiple echo statements (e.g. `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). An attacker who controls the `sbt-runner-version` input can inject newlines into $GITHUB_OUTPUT to set arbitrary output variables or environment values for subsequent steps.

Locations:

- `action.yml:26`
- `action.yml:30`
- `action.yml:35`
- `action.yml:40`

### script-injection (severity: high)

Rule (b) violation: In the 'Download and Install sbt' step, the env var `$SBT_RUNNER_VERSION` (sourced from the user-controlled `inputs.sbt-runner-version`) is expanded unquoted inside double-quoted URL strings passed to curl: `curl -sL "https://github.com/sbt/sbt/releases/download/v$SBT_RUNNER_VERSION/sbt-$SBT_RUNNER_VERSION.zip"`. Although the outer string is double-quoted, the variable is not separately quoted and a value containing shell metacharacters (e.g. `$(command)`, backticks, or whitespace) can still be interpreted by the shell, enabling command injection.

Locations:

- `action.yml:76`
- `action.yml:78`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed both findings in action.yml:

1. github-env-injection (lines 26, 30, 35, 40): Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the 'Set up cache paths' run block. Replaced all uses of `$SBT_RUNNER_VERSION` in `$GITHUB_OUTPUT` echo statements with `$SAFE_SBT_RUNNER_VERSION` to prevent newline injection attacks.

2. script-injection (lines 76, 78): Added the same sanitization in the 'Download and Install sbt' run block. Replaced all uses of `$SBT_RUNNER_VERSION` in curl URLs and file paths with `${SAFE_SBT_RUNNER_VERSION}` (using brace syntax for clarity) to prevent shell metacharacter injection.

