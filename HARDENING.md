<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.10** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the untrusted input `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION` and then writes it to `$GITHUB_OUTPUT` multiple times without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A caller-controlled newline in the version string could inject arbitrary key=value pairs into the GitHub Actions output context. Additionally, the 'Setup PATH' step writes `$PWD/sbt/bin` to `$GITHUB_PATH` after `cd "$SBT_TOOLPATH"` where `SBT_TOOLPATH` is derived from the same untrusted input, without sanitization — an attacker-controlled newline in the version string could inject arbitrary entries into `$GITHUB_PATH`.

Locations:

- `action.yml:27`
- `action.yml:36`
- `action.yml:44`
- `action.yml:52`
- `action.yml:55`
- `action.yml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in action.yml:
1. 'Set up cache paths' step: Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script. Replaced all occurrences of `$SBT_RUNNER_VERSION` in $GITHUB_OUTPUT writes (sbt_toolpath on Windows/macOS/Linux, and sbt_cachekey) with `$SAFE_SBT_RUNNER_VERSION` to prevent newline injection from the untrusted `inputs.sbt-runner-version` value.
2. 'Setup PATH' step: Added sanitization using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows equivalent with backslashes) before writing to $GITHUB_PATH, preventing newline injection via the attacker-controlled path derived from the version input.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Download and Install sbt' step in action.yml: added sanitization of SBT_RUNNER_VERSION at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`, then replaced all uses of the raw `$SBT_RUNNER_VERSION` with `$SAFE_SBT_RUNNER_VERSION` in both curl URLs, file paths, and the unzip command. This prevents shell command injection via the attacker-controllable `sbt-runner-version` input.

