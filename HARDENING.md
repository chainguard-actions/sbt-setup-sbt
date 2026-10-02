<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.1** was hardened automatically. 1 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from inputs.sbt-runner-version (an attacker-controlled value) and then written to $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'). A caller supplying a version string containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps. Affected lines include the echo statements writing sbt_toolpath (Windows, macOS, and Linux branches) and sbt_cachekey.

Locations:

- `action.yml:26`
- `action.yml:31`
- `action.yml:36`
- `action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of SBT_RUNNER_VERSION at the start of the 'Set up cache paths' step's run script. The attacker-controlled input is now sanitized via `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before being written to $GITHUB_OUTPUT. All four affected echo statements (Windows sbt_toolpath, macOS sbt_toolpath, Linux sbt_toolpath, and sbt_cachekey) now use the sanitized variable, preventing newline injection attacks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in the 'Download and Install sbt' step of action.yml. Added sanitization of SBT_RUNNER_VERSION at the start of the run block using `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` to produce SAFE_SBT_RUNNER_VERSION, then replaced all unquoted uses of $SBT_RUNNER_VERSION in curl URLs, file paths, and the unzip command with properly quoted ${SAFE_SBT_RUNNER_VERSION}. This prevents shell metacharacter injection from attacker-controlled version strings.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml (around line 160) by sanitizing $PWD before writing to $GITHUB_PATH. Added `SAFE_PWD=$(printf '%s' "$PWD" | tr -d '\n\r')` and replaced `$PWD` with `${SAFE_PWD}` in both the Windows and non-Windows branches. This prevents an attacker-controlled `inputs.sbt-runner-version` containing newline characters from injecting arbitrary entries into $GITHUB_PATH.

