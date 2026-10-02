<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version) is written to $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). An attacker-controlled version string containing embedded newlines could inject arbitrary key-value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps. Affected lines include the sbt_toolpath writes (Windows, macOS, and Linux branches) and the sbt_cachekey write.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

In the 'Setup PATH' step, $PWD/sbt/bin (where $PWD is derived from SBT_TOOLPATH, which is set from steps.cache-paths.outputs.sbt_toolpath — itself built from the caller-controlled inputs.sbt-runner-version) is written to $GITHUB_PATH without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A newline-containing version input could inject an arbitrary directory into $GITHUB_PATH, enabling PATH hijacking.

Locations:

- `action.yml:155`
- `action.yml:157`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (lines 27, 32, 37, 42): Added sanitization of SBT_RUNNER_VERSION at the start of the run block using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four GITHUB_OUTPUT writes that included the version string (sbt_toolpath on Windows, macOS, and Linux, plus sbt_cachekey) now use the sanitized variable.

2. 'Setup PATH' step (lines 155, 157): Added sanitization of SBT_TOOLPATH using `SAFE_SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` and used it for the cd command. Each branch (Windows and non-Windows) now sanitizes the computed path via `safe_path=$(printf '%s' "..." | tr -d '\n\r')` before writing to $GITHUB_PATH.

