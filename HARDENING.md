<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.8** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the input `inputs.sbt-runner-version` is mapped to the env var `SBT_RUNNER_VERSION` and then written unsanitized to `$GITHUB_OUTPUT` in multiple echo statements (e.g., `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`). An attacker-controlled value containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. The required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent before every write.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:39`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added a sanitization line `SBT_RUNNER_VERSION_SAFE=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script, then replaced all occurrences of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT writes with `$SBT_RUNNER_VERSION_SAFE`. This prevents newline injection attacks via the `inputs.sbt-runner-version` input.

