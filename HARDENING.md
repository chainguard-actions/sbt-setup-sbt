<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.6** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the input `inputs.sbt-runner-version` is mapped to the env var `SBT_RUNNER_VERSION` and then written unsanitized into `$GITHUB_OUTPUT` on multiple lines — e.g.:
  `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`
  `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`
A caller-controlled value containing newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs. The required sanitization (`safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`) is absent before every write.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`
- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in the 'Set up cache paths' step of action.yml. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, then replaced all uses of `$SBT_RUNNER_VERSION` in `$GITHUB_OUTPUT` writes with `$SAFE_SBT_RUNNER_VERSION`. This prevents newline injection via the caller-controlled `inputs.sbt-runner-version` value across all 5 affected write locations (the three OS-specific sbt_toolpath writes and the sbt_cachekey write).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml to sanitize the path before writing to $GITHUB_PATH. Instead of directly echoing '$PWD\\sbt\\bin' or '$PWD/sbt/bin' to $GITHUB_PATH, the fix captures the path into a variable using `printf '%s' "$PWD/..." | tr -d '\n\r'` to strip any embedded newline/carriage-return characters, then writes the sanitized value. This prevents a malicious sbt_toolpath output containing newlines from injecting arbitrary entries into $GITHUB_PATH.

