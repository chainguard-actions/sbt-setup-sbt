<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `${{ inputs.sbt-runner-version }}` (an attacker-controlled input) and then written directly into `$GITHUB_OUTPUT` without sanitization. Multiple lines write values containing `$SBT_RUNNER_VERSION` to `$GITHUB_OUTPUT`, e.g.:
  `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`
  `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`
The required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent before each write. A newline-containing version string could inject arbitrary key=value pairs into the output context.

Additionally, the 'Setup PATH' step writes `$PWD/sbt/bin` to `$GITHUB_PATH` where `$PWD` is derived from `$SBT_TOOLPATH`, which was set from the tainted `sbt_toolpath` output (itself containing the unsanitized `inputs.sbt-runner-version`). This write also lacks sanitization, allowing a malicious version string to inject arbitrary entries into `$GITHUB_PATH`.

Locations:

- `action.yml:26`
- `action.yml:31`
- `action.yml:38`
- `action.yml:43`
- `action.yml:44`
- `action.yml:232`
- `action.yml:234`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in two locations in action.yml:
1. 'Set up cache paths' step: Added sanitization of SBT_RUNNER_VERSION via `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script, then replaced all $SBT_RUNNER_VERSION references in $GITHUB_OUTPUT writes with $SAFE_SBT_RUNNER_VERSION.
2. 'Setup PATH' step: Added sanitization of the path string before writing to $GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows equivalent), then writing $safe_path instead of the raw value.

