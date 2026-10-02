<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets the env var SBT_RUNNER_VERSION from the user-controlled input `inputs.sbt-runner-version` and then writes it into $GITHUB_OUTPUT multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who supplies a version string containing a newline character can inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps. Affected lines include:
  - `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows branch, ~line 27)
  - `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (macOS branch, ~line 32)
  - `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Linux branch, ~line 37)
  - `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"` (~line 42)

Fix: sanitize the value before each write, e.g.:
  safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')
  echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$safe" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added sanitization of the user-controlled SBT_RUNNER_VERSION input before writing it to $GITHUB_OUTPUT. Added `safe_sbt_runner_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, then replaced all four uses of $SBT_RUNNER_VERSION in echo-to-GITHUB_OUTPUT statements (Windows sbt_toolpath, macOS sbt_toolpath, Linux sbt_toolpath, and sbt_cachekey) with $safe_sbt_runner_version.

