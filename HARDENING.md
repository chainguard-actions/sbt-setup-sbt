<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.8** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from inputs.sbt-runner-version (an attacker-controllable input) and is written to $GITHUB_OUTPUT multiple times without sanitization. For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A newline character embedded in the input value could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`) is absent before each write.

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added sanitization of the attacker-controllable SBT_RUNNER_VERSION input before writing it to $GITHUB_OUTPUT. A new variable SAFE_SBT_RUNNER_VERSION is computed at the start of the run script using `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`, and all subsequent $GITHUB_OUTPUT writes that included the version (sbt_toolpath in all three OS branches, and sbt_cachekey) now use $SAFE_SBT_RUNNER_VERSION instead of $SBT_RUNNER_VERSION. This prevents newline injection attacks via the sbt-runner-version input.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml (around line 228) by sanitizing the path before writing to $GITHUB_PATH. The fix uses `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and the Windows equivalent) to strip newline and carriage return characters before the `echo "$safe_path" >> "$GITHUB_PATH"` write. This prevents newline injection attacks where an attacker controlling the `sbt_toolpath` step output could inject arbitrary entries into $GITHUB_PATH.

