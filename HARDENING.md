<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from inputs.sbt-runner-version via its env: block, then writes values derived from that input directly into $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). An attacker-controlled value for inputs.sbt-runner-version containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps.

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added sanitization of the user-controlled SBT_RUNNER_VERSION input at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All subsequent writes to $GITHUB_OUTPUT that included the version string now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable instead, preventing newline injection attacks that could overwrite downstream step outputs.

