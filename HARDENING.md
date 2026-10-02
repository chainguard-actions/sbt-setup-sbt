<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is sourced from inputs.sbt-runner-version (an untrusted caller-controlled input) and is written directly to $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). An attacker-controlled version string containing newlines could inject arbitrary key=value pairs into the GitHub output context. Affected lines write values like: echo "sbt_toolpath=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT" and echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT".

Locations:

- `action.yml:23`
- `action.yml:27`
- `action.yml:31`
- `action.yml:35`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is sourced from steps.cache-paths.outputs.sbt_toolpath (a step output that was itself derived from inputs.sbt-runner-version). The step does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization. Because $PWD is derived from the untrusted input, a newline-containing value could inject additional entries into $GITHUB_PATH. The write `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` is missing the required `printf '%s' | tr -d '\n\r'` sanitization.

Locations:

- `action.yml:200`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/action.yml:

1. 'Set up cache paths' step (lines 23, 27, 31, 35): Added sanitization of the caller-controlled SBT_RUNNER_VERSION input at the top of the run script using `SAFE_SBT_RUNNER_VERSION="$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')"`. All GITHUB_OUTPUT writes that included the version string now use the sanitized variable.

2. 'Setup PATH' step (line 200): Added sanitization of the $PWD-derived path before writing to $GITHUB_PATH. Both the Windows and non-Windows branches now use `SAFE_PATH="$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')"` before echoing to $GITHUB_PATH.

