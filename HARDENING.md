<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.5** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is set from inputs.sbt-runner-version (untrusted caller-controlled input) and is written directly into $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' | tr -d '\n\r'). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. A malicious version string containing embedded newlines could inject additional key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:39`
- `action.yml:40`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is set from steps.cache-paths.outputs.sbt_toolpath, which was itself derived from the untrusted input inputs.sbt-runner-version. The step does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization (no printf '%s' | tr -d '\n\r'). Because SBT_TOOLPATH is tainted by the caller-controlled input, the value appended to GITHUB_PATH is also tainted and could contain injected newlines that add attacker-controlled entries to the runner's PATH.

Locations:

- `action.yml:196`
- `action.yml:198`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/action.yml:

1. 'Set up cache paths' step (lines 27-40): Added sanitization of SBT_RUNNER_VERSION at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All GITHUB_OUTPUT writes that included the version string now use $SAFE_SBT_RUNNER_VERSION instead of $SBT_RUNNER_VERSION.

2. 'Setup PATH' step (lines 196-198): Added sanitization before writing to GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and the Windows equivalent with backslashes), then writing `$safe_path` to GITHUB_PATH instead of the raw path value. This prevents the tainted SBT_TOOLPATH (derived from the untrusted sbt-runner-version input) from injecting additional PATH entries via embedded newlines.

