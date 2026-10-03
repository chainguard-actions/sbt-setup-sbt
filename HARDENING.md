<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets the env var SBT_RUNNER_VERSION from the caller-controlled input `inputs.sbt-runner-version`, then writes it into $GITHUB_OUTPUT multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who controls the `sbt-runner-version` input can inject newlines to poison subsequent GITHUB_OUTPUT entries or other environment files. Affected lines include: `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows, macOS, and Linux branches) and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of the caller-controlled `SBT_RUNNER_VERSION` variable at the start of the 'Set up cache paths' run script. The line `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` strips newline and carriage-return characters before the variable is used in any `echo ... >> "$GITHUB_OUTPUT"` statements (lines 27, 32, 37, and 42). This prevents newline injection attacks that could poison subsequent GITHUB_OUTPUT entries.

