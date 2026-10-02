<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.8** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `inputs.sbt-runner-version` (caller-controlled) and then written unsanitized to `$GITHUB_OUTPUT` in multiple echo statements — for example:
  `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`
  `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`
None of these writes are preceded by the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller supplying a version string containing embedded newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs consumed by later steps.

Locations:

- `action.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added sanitization of the caller-controlled `SBT_RUNNER_VERSION` input at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All subsequent writes to $GITHUB_OUTPUT that included the version string now use `$SAFE_SBT_RUNNER_VERSION` instead of the unsanitized `$SBT_RUNNER_VERSION`, preventing newline injection attacks that could overwrite arbitrary output key=value pairs.

