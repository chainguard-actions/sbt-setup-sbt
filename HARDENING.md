<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step writes the untrusted input `inputs.sbt-runner-version` (via the env var `SBT_RUNNER_VERSION`) directly into `$GITHUB_OUTPUT` without sanitization. Multiple `echo` lines such as `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"` are affected. A caller supplying a version string containing embedded newline characters could inject arbitrary additional key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps. The required sanitization step (`safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`) is absent before every write.

Locations:

- `action.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added sanitization of the SBT_RUNNER_VERSION input by computing `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script. All echo commands that write to $GITHUB_OUTPUT now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable instead of the raw `$SBT_RUNNER_VERSION`, preventing newline injection attacks that could overwrite downstream step outputs.

