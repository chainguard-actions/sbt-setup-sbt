<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the untrusted input `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION` and then writes it directly to `$GITHUB_OUTPUT` on multiple lines without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). An attacker who controls the `sbt-runner-version` input can inject newlines into `$GITHUB_OUTPUT`, potentially overwriting subsequent output variables and influencing downstream steps. Affected writes include:
- `echo "sbt_toolpath=...\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows, macOS, Linux branches, lines ~27/32/37)
- `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"` (line ~42)
None of these writes are preceded by the required sanitization pipeline.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added sanitization at the start of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four $GITHUB_OUTPUT writes that previously used the raw `$SBT_RUNNER_VERSION` (sbt_toolpath on Windows/macOS/Linux branches, and sbt_cachekey) now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable instead, preventing newline injection attacks.

