<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the user-controlled input `inputs.sbt-runner-version` to the env var `SBT_RUNNER_VERSION`, then writes it unsanitized to `$GITHUB_OUTPUT` in multiple echo statements (e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). No sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is applied before any of these writes. An attacker who controls the `sbt-runner-version` input can inject newlines to poison subsequent steps' outputs or environment via GITHUB_OUTPUT.

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the run script to sanitize the user-controlled `sbt-runner-version` input before it is used in any echo statements that write to $GITHUB_OUTPUT. This prevents newline injection attacks that could poison subsequent steps' outputs or environment variables.

