<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets the env var SBT_RUNNER_VERSION from the caller-controlled input `${{ inputs.sbt-runner-version }}` and then writes it unsanitized to $GITHUB_OUTPUT multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. Because no `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` sanitization is applied before these writes, an attacker who supplies a version string containing newline characters can inject arbitrary key=value pairs into $GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps.

Locations:

- `action.yml:18`
- `action.yml:26`
- `action.yml:31`
- `action.yml:36`
- `action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in the 'Set up cache paths' step of action.yml. Added sanitization of the caller-controlled SBT_RUNNER_VERSION input by computing SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r') at the start of the run script. All five $GITHUB_OUTPUT writes that previously used $SBT_RUNNER_VERSION (sbt_toolpath on Windows, sbt_toolpath on macOS, sbt_toolpath on Linux, sbt_cachekey) now use $SAFE_SBT_RUNNER_VERSION instead, preventing newline injection attacks.

