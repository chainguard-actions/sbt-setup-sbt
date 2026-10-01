<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `${{ inputs.sbt-runner-version }}` (a caller-controlled input) and then written unsanitized to `$GITHUB_OUTPUT` multiple times — for example:
  `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`
  `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`
No `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` sanitization is applied before any of these writes. A caller supplying a version string containing embedded newlines (e.g. `2.0.3\nFOO=injected`) could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps.

Locations:

- `action.yml:18`
- `action.yml:25`
- `action.yml:30`
- `action.yml:35`
- `action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the 'Set up cache paths' run script in action.yml. This sanitizes the caller-controlled input before it is used in any `echo ... >> "$GITHUB_OUTPUT"` writes (sbt_toolpath, sbt_cachekey, etc.), preventing newline injection attacks that could overwrite arbitrary GITHUB_OUTPUT key=value pairs.

