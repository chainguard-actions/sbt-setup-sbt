<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Set up cache paths' writes the value of $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version via env:) to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Multiple echo lines write this unsanitized input-derived value, e.g.: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A newline injected into inputs.sbt-runner-version would allow an attacker to inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:27`

### github-env-injection (severity: high)

Step 'Setup PATH' writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization. $PWD is derived from `cd "$SBT_TOOLPATH"` where $SBT_TOOLPATH comes from steps.cache-paths.outputs.sbt_toolpath, which is itself derived from the user-controlled input inputs.sbt-runner-version. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write: `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`. A newline in the input could inject an arbitrary path entry into GITHUB_PATH.

Locations:

- `action.yml:183`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step (line 27): Added sanitization of SBT_RUNNER_VERSION using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script. All GITHUB_OUTPUT writes that included the version string now use $SAFE_SBT_RUNNER_VERSION instead of $SBT_RUNNER_VERSION.
2. 'Setup PATH' step (line 183): Added sanitization of the path before writing to GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and the Windows equivalent), then writing $safe_path to $GITHUB_PATH instead of the raw path.

