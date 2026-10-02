<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the input `inputs.sbt-runner-version` is assigned to the env var `SBT_RUNNER_VERSION` and then written unsanitized to `$GITHUB_OUTPUT` in multiple places — embedded in path strings such as `sbt_toolpath=...$SBT_RUNNER_VERSION` (lines ~27, 32, 37) and `sbt_cachekey=...-$SBT_RUNNER_VERSION-...` (line ~42). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller supplying a version string containing newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs consumed by later steps. This is a case-(d) violation (indirect write of inputs via env var without sanitization).

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step, added sanitization of SBT_RUNNER_VERSION at the top of the run script: `SBT_RUNNER_VERSION_SAFE=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. Replaced all four uses of `$SBT_RUNNER_VERSION` in $GITHUB_OUTPUT writes (sbt_toolpath for Windows, macOS, and Linux branches, and sbt_cachekey) with `$SBT_RUNNER_VERSION_SAFE` to prevent newline injection attacks.

