<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sbt--setup-sbt/v1.1.23** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is set from the untrusted input `${{ inputs.sbt-runner-version }}` and then written directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-controlled version string containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. Affected lines: `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (lines 19 and 22) and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"` (line 25).

Locations:

- `action.yml:19`
- `action.yml:22`
- `action.yml:25`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in the 'Set up cache paths' step of action.yml. Added a sanitization line `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, then replaced all three occurrences of `$SBT_RUNNER_VERSION` in the GITHUB_OUTPUT echo commands (the sbt_toolpath on Windows, sbt_toolpath on Linux/macOS, and sbt_cachekey outputs) with `$safe_version`. This prevents newline injection attacks where an attacker-controlled version string could inject arbitrary key=value pairs into GITHUB_OUTPUT.

