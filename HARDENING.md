<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `inputs.sbt-runner-version` (a caller-controlled input) and then written unsanitized to `$GITHUB_OUTPUT` in multiple echo statements (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`). If an attacker supplies a version string containing embedded newlines, they can inject arbitrary `key=value` pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs consumed by later steps. The required sanitization step (`safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`) is absent before every write.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of SBT_RUNNER_VERSION at the start of the 'Set up cache paths' run script. The caller-controlled input is now stripped of newlines and carriage returns via `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before being used in any `$GITHUB_OUTPUT` writes. All four echo statements that previously used `$SBT_RUNNER_VERSION` in output writes (sbt_toolpath on Windows, sbt_toolpath on macOS, sbt_toolpath on Linux, and sbt_cachekey) now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml (line ~211) to sanitize the path value before writing to $GITHUB_PATH. The fix uses `safe=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` and then `echo "$safe" >> "$GITHUB_PATH"` for both the Windows and non-Windows branches. This prevents any embedded newlines in the `steps.cache-paths.outputs.sbt_toolpath` step output from injecting arbitrary entries into $GITHUB_PATH.

