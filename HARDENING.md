<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the input `inputs.sbt-runner-version` is loaded into the env var `SBT_RUNNER_VERSION` and then written to `$GITHUB_OUTPUT` multiple times without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A caller-controlled version string containing newlines could inject arbitrary key=value pairs into the output context. Affected writes include: `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (three OS branches) and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:38`
- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of SBT_RUNNER_VERSION at the start of the 'Set up cache paths' run script using `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. This strips newlines and carriage returns from the caller-controlled version string before it is written to $GITHUB_OUTPUT in four places (sbt_toolpath on all three OS branches, and sbt_cachekey). The env block already correctly loads the expression into an env var; the sanitization step ensures no newline injection can occur when writing to the output context.

