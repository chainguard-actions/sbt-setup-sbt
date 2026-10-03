<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step in action.yml maps the user-controlled input `inputs.sbt-runner-version` to the env var `SBT_RUNNER_VERSION`, then writes it unsanitized into `$GITHUB_OUTPUT` multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. No sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is applied before any of these writes. A caller supplying a version string containing embedded newlines can inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs or poisoning the environment for downstream steps.

Locations:

- `action.yml:15`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added a sanitization line at the top of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All GITHUB_OUTPUT writes that previously used `$SBT_RUNNER_VERSION` (sbt_toolpath and sbt_cachekey) now use `$SAFE_SBT_RUNNER_VERSION` instead, preventing newline injection attacks via the user-controlled `inputs.sbt-runner-version` value.

