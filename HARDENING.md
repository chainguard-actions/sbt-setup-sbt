<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the input `inputs.sbt-runner-version` is assigned to the env var `SBT_RUNNER_VERSION` (via `env: SBT_RUNNER_VERSION: ${{ inputs.sbt-runner-version }}`), and then written unsanitized to `$GITHUB_OUTPUT` multiple times — e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. No `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` sanitization step is applied before any of these writes. A caller-controlled value containing newlines could inject arbitrary key=value pairs into the GitHub Actions output context.

Locations:

- `action.yml:19`
- `action.yml:27`
- `action.yml:31`
- `action.yml:36`
- `action.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of SBT_RUNNER_VERSION at the start of the 'Set up cache paths' step's run script. The line `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` strips newlines and carriage returns from the caller-controlled input before it is written to $GITHUB_OUTPUT in multiple places (sbt_toolpath, sbt_cachekey). This prevents a malicious caller from injecting arbitrary key=value pairs into the GitHub Actions output context via embedded newlines.

