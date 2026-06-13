<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sbt--setup-sbt/v1.2.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from inputs.sbt-runner-version (an untrusted caller-controlled input) via the env: block, then writes it unsanitized into $GITHUB_OUTPUT multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A newline embedded in the version string could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps. Additionally, the 'Setup PATH' step writes `$PWD/sbt/bin` to $GITHUB_PATH after `cd "$SBT_TOOLPATH"`, where SBT_TOOLPATH is derived from steps.cache-paths.outputs.sbt_toolpath (itself built from the untrusted input). No sanitization is applied before the GITHUB_PATH write either.

Locations:

- `action.yml:26`
- `action.yml:31`
- `action.yml:36`
- `action.yml:41`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in action.yml:

1. 'Set up cache paths' step: Added sanitization of the caller-controlled SBT_RUNNER_VERSION input using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block. All GITHUB_OUTPUT writes that included the version string now use $SAFE_SBT_RUNNER_VERSION instead of $SBT_RUNNER_VERSION, preventing newline injection.

2. 'Setup PATH' step: Added sanitization of SBT_TOOLPATH (derived from the untrusted input) using `SAFE_SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` before the cd command. The path values written to GITHUB_PATH are also sanitized with `safe_path=$(printf '%s' "..." | tr -d '\n\r')` for both Windows and non-Windows branches.

