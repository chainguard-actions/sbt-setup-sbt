<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is sourced from inputs.sbt-runner-version (an attacker-controlled composite-action input) and written directly to $GITHUB_OUTPUT on multiple lines without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). A malicious version string containing embedded newlines could inject arbitrary key-value pairs into the output context, potentially poisoning downstream steps. Affected lines include all three 'sbt_toolpath=...$SBT_RUNNER_VERSION' writes (Windows, macOS, Linux branches) and the 'sbt_cachekey=...$SBT_RUNNER_VERSION...' write.

Locations:

- `action.yml:26`
- `action.yml:31`
- `action.yml:36`
- `action.yml:41`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is sourced from steps.cache-paths.outputs.sbt_toolpath (which was itself derived from the unsanitized inputs.sbt-runner-version). After 'cd "$SBT_TOOLPATH"', the script writes $PWD to $GITHUB_PATH without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker-controlled version string that manipulates the tool path could inject additional entries into the runner's PATH.

Locations:

- `action.yml:201`
- `action.yml:203`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step (lines 26, 31, 36, 41): Added sanitization of SBT_RUNNER_VERSION at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four GITHUB_OUTPUT writes that included the version string (sbt_toolpath for Windows/macOS/Linux, and sbt_cachekey) now use the sanitized variable.
2. 'Setup PATH' step (lines 201, 203): Added sanitization of PWD using `SAFE_PWD=$(printf '%s' "$PWD" | tr -d '\n\r')` before writing to GITHUB_PATH. Both the Windows and non-Windows branches now use $SAFE_PWD instead of $PWD.

