<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is sourced from inputs.sbt-runner-version (an untrusted caller-controlled input) and is written unsanitized into $GITHUB_OUTPUT multiple times. For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. None of these writes are preceded by the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). An attacker who controls the sbt-runner-version input could inject newlines to poison GITHUB_OUTPUT with arbitrary key=value pairs, potentially overwriting outputs consumed by later steps.

Locations:

- `action.yml:27`
- `action.yml:33`
- `action.yml:39`
- `action.yml:45`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in the 'Set up cache paths' step of action.yml. Added sanitization at the start of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. Replaced all four occurrences of `$SBT_RUNNER_VERSION` in `$GITHUB_OUTPUT` writes (sbt_toolpath on Windows, macOS, and Linux branches, plus sbt_cachekey) with the sanitized `$SAFE_SBT_RUNNER_VERSION` variable. This prevents an attacker-controlled sbt-runner-version input from injecting newlines to poison GITHUB_OUTPUT with arbitrary key=value pairs.

