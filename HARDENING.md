<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.4** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from inputs.sbt-runner-version (a caller-controlled value) and then written unsanitized into $GITHUB_OUTPUT on multiple lines — e.g.:
  echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"
  echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"
No sanitization step (printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r') is applied before these writes. An attacker who controls the sbt-runner-version input can inject newlines to poison $GITHUB_OUTPUT with arbitrary key=value pairs, potentially overwriting outputs consumed by later steps. This is a case (d) violation: indirect write of an input via env var without sanitization.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:38`
- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the run script to sanitize the caller-controlled sbt-runner-version input before it is written to $GITHUB_OUTPUT on lines 27, 32, 38, and 43. This strips all newline and carriage return characters, preventing an attacker from injecting newlines to poison $GITHUB_OUTPUT with arbitrary key=value pairs.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml by sanitizing the SBT_TOOLPATH environment variable before using it. Added `SAFE_SBT_TOOLPATH="$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')"` before the `cd` command, and changed `cd "$SBT_TOOLPATH"` to `cd "$SAFE_SBT_TOOLPATH"`. This ensures that any newlines injected into the steps.cache-paths.outputs.sbt_toolpath value are stripped before the path is used, preventing an attacker from injecting additional entries into $GITHUB_PATH.

