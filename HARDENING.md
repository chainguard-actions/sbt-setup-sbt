<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets the env var SBT_RUNNER_VERSION from the user-controlled input `${{ inputs.sbt-runner-version }}` and then writes it unsanitized into $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. An attacker who controls the `sbt-runner-version` input could inject newlines to poison subsequent step outputs or environment variables.

Locations:

- `action.yml:19`
- `action.yml:26`
- `action.yml:31`
- `action.yml:36`
- `action.yml:41`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added sanitization of the user-controlled SBT_RUNNER_VERSION input at the start of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four occurrences where the version was written to $GITHUB_OUTPUT (sbt_toolpath on Windows/macOS/Linux, and sbt_cachekey) now use $SAFE_SBT_RUNNER_VERSION instead of the raw $SBT_RUNNER_VERSION, preventing newline injection attacks.

