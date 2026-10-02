<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the user-controlled input `inputs.sbt-runner-version` to the env var `SBT_RUNNER_VERSION` and then writes it to `$GITHUB_OUTPUT` multiple times without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker supplying a newline-containing version string could inject arbitrary key=value pairs into the GitHub output context. Affected lines include the `sbt_toolpath` writes (lines 27, 32, 37) and the `sbt_cachekey` write (line 42), all of the form: `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`. This is a case-(d) violation: an untrusted input is routed through an env var and written to a special environment file without sanitization.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added sanitization of the user-controlled `SBT_RUNNER_VERSION` env var at the start of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four writes to $GITHUB_OUTPUT that included the version string (sbt_toolpath on Windows, macOS, and Linux branches, plus sbt_cachekey) now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable instead of the raw `$SBT_RUNNER_VERSION`, preventing newline injection attacks.

