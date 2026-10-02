<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets the env var `SBT_RUNNER_VERSION` from `inputs.sbt-runner-version` (an untrusted caller-controlled input) and then writes it unsanitized into `$GITHUB_OUTPUT` via multiple `echo` statements (e.g. `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. An attacker supplying a newline-containing version string could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script to sanitize the caller-controlled `sbt-runner-version` input. Replaced all 4 occurrences of `$SBT_RUNNER_VERSION` in `$GITHUB_OUTPUT` writes (lines 27, 32, 37, 41) with `$SAFE_SBT_RUNNER_VERSION` to prevent newline injection attacks.

