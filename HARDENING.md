<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the untrusted input `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION` and then writes it unsanitized to `$GITHUB_OUTPUT` on multiple lines — e.g. `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller-supplied version string containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps. The required fix is to sanitize the value before use: `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and then use `$safe` in the echo statements.

Locations:

- `action.yml:24`
- `action.yml:30`
- `action.yml:36`
- `action.yml:44`
- `action.yml:45`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the 'Set up cache paths' run block, then replaced all uses of `$SBT_RUNNER_VERSION` in echo-to-GITHUB_OUTPUT statements with `$safe`. This sanitizes the untrusted `inputs.sbt-runner-version` value before it is written to GITHUB_OUTPUT, preventing newline injection attacks that could overwrite outputs consumed by later steps.

