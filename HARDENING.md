<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the user-controlled input `inputs.sbt-runner-version` to the env var `SBT_RUNNER_VERSION`, then writes it directly into `$GITHUB_OUTPUT` on multiple lines without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). An attacker who controls the `sbt-runner-version` input can embed newline characters to inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning downstream steps. Affected lines include: `echo "sbt_toolpath=...\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (line 26, 31, 37) and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"` (line 42). The same pattern is repeated in the 'Download and Install sbt' step (env: SBT_RUNNER_VERSION used unsanitized in curl URL construction, line ~100).

Locations:

- `action.yml:18`
- `action.yml:26`
- `action.yml:31`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of both affected run blocks in action.yml: (1) the 'Set up cache paths' step (lines 26, 31, 37, 42) where SBT_RUNNER_VERSION was written unsanitized to $GITHUB_OUTPUT in sbt_toolpath and sbt_cachekey values, and (2) the 'Download and Install sbt' step (~line 100) where SBT_RUNNER_VERSION was used unsanitized in curl URL construction. The sanitization strips newline and carriage return characters from the user-controlled input before it is used anywhere, preventing GITHUB_OUTPUT injection attacks.

