<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.23** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the user-controlled input `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION`, then writes it unsanitized into `$GITHUB_OUTPUT` on multiple lines:
  - `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`
  - `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`

No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller supplying a newline-containing version string could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs consumed by later steps. The required fix is to sanitize the value before writing, e.g.:
  `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`
and then use `$safe` in the echo statements.

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set up cache paths' step in action.yml by sanitizing the user-controlled `inputs.sbt-runner-version` value before writing it to $GITHUB_OUTPUT. Added `safe_sbt_runner_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and replaced all `$SBT_RUNNER_VERSION` references in the GITHUB_OUTPUT echo statements with `$safe_sbt_runner_version`. This prevents newline injection attacks that could allow callers to inject arbitrary key=value pairs into $GITHUB_OUTPUT.

