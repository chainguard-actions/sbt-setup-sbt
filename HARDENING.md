<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version via the step's env: block) is written to $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A caller supplying a newline-containing value for sbt-runner-version can inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially hijacking subsequent step outputs. Additionally, the 'Setup PATH' step writes $PWD (derived from `cd "$SBT_TOOLPATH"` where SBT_TOOLPATH is transitively derived from inputs.sbt-runner-version) to $GITHUB_PATH without sanitization: `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`, allowing PATH injection.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`
- `action.yml:207`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in action.yml:
1. 'Set up cache paths' step: Added sanitization of SBT_RUNNER_VERSION using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script. All GITHUB_OUTPUT writes that included the version now use $SAFE_SBT_RUNNER_VERSION instead of $SBT_RUNNER_VERSION.
2. 'Setup PATH' step: Replaced direct writes of $PWD-derived paths to $GITHUB_PATH with sanitized versions using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` before writing to $GITHUB_PATH, for both Windows and non-Windows branches.

