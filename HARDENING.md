<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the caller-controlled input `inputs.sbt-runner-version` and then written unsanitized to $GITHUB_OUTPUT multiple times (e.g. `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller supplying a version string containing newline characters could inject arbitrary key=value pairs into $GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step, added sanitization of SBT_RUNNER_VERSION at the top of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four writes to $GITHUB_OUTPUT that included the version string (sbt_toolpath on Windows, sbt_toolpath on macOS, sbt_toolpath on Linux, and sbt_cachekey) now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable instead of the raw `$SBT_RUNNER_VERSION`, preventing newline injection attacks via the caller-controlled input.

