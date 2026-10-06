<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.11** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is set from the caller-controlled input `inputs.sbt-runner-version` and then written to $GITHUB_OUTPUT multiple times without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). A caller can supply a version string containing newline characters to inject arbitrary key=value pairs into $GITHUB_OUTPUT, poisoning outputs consumed by later steps. Affected lines include the Windows branch (`echo "sbt_toolpath=...\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`), the macOS branch, the Linux/else branch, and the final `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"` line. None of these writes are preceded by the sanitization pipeline.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the 'Set up cache paths' run script. This sanitizes the caller-controlled `inputs.sbt-runner-version` value (already moved into the env block as SBT_RUNNER_VERSION) by stripping newline and carriage return characters before it is used in any of the four $GITHUB_OUTPUT writes (Windows sbt_toolpath, macOS sbt_toolpath, Linux/else sbt_toolpath, and sbt_cachekey). This prevents newline injection attacks that could poison outputs consumed by later steps.

