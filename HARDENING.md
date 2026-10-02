<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version) is written directly to $GITHUB_OUTPUT on multiple lines without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker-controlled input value containing newlines could inject arbitrary key=value pairs into the GitHub output context. Affected lines include: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows branch, line 27), the macOS branch (line 32), the Linux branch (line 37), and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"` (line 42).

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

In the 'Setup PATH' step, $PWD (derived from `cd "$SBT_TOOLPATH"` where $SBT_TOOLPATH is set from steps.cache-paths.outputs.sbt_toolpath, which was itself constructed from the untrusted inputs.sbt-runner-version) is written to $GITHUB_PATH without sanitization. A malicious sbt-runner-version input containing newlines could inject arbitrary entries into $GITHUB_PATH. The offending lines are: `echo "$PWD\\sbt\\bin" >> "$GITHUB_PATH"` (Windows branch) and `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` (Linux/macOS branch).

Locations:

- `action.yml:214`
- `action.yml:216`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step (lines 27, 32, 37, 42): Added sanitization of SBT_RUNNER_VERSION via `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before the OS-conditional block, and replaced all $SBT_RUNNER_VERSION references in GITHUB_OUTPUT writes with $SAFE_SBT_RUNNER_VERSION.
2. 'Setup PATH' step (lines 214, 216): Replaced direct echo of $PWD-based paths to $GITHUB_PATH with sanitized versions using `safe_path=$(printf '%s' "$PWD/..." | tr -d '\n\r')` before writing to $GITHUB_PATH.

