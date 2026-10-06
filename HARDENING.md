<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from inputs.sbt-runner-version (an untrusted caller-controlled input) via the env: block, then writes it unsanitized to $GITHUB_OUTPUT in multiple echo statements (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A newline-containing version string could inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:20`

### github-env-injection (severity: high)

The 'Setup PATH' step sets SBT_TOOLPATH from steps.cache-paths.outputs.sbt_toolpath (a step output derived from the untrusted inputs.sbt-runner-version) via the env: block. The script executes `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH (e.g., `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). Since $PWD is derived from the untrusted $SBT_TOOLPATH, this constitutes an unsanitized write of attacker-controlled data to $GITHUB_PATH. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write.

Locations:

- `action.yml:196`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (line 20): Added sanitization of SBT_RUNNER_VERSION using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before all GITHUB_OUTPUT writes. Replaced all occurrences of $SBT_RUNNER_VERSION in echo-to-GITHUB_OUTPUT statements with $SAFE_SBT_RUNNER_VERSION.

2. 'Setup PATH' step (line 196): Added sanitization of $PWD (which is derived from the untrusted SBT_TOOLPATH input) using `SAFE_PWD=$(printf '%s' "$PWD" | tr -d '\n\r')` before the GITHUB_PATH write. Replaced $PWD with $SAFE_PWD in the echo-to-GITHUB_PATH statement.

