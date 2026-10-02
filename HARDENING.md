<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from inputs.sbt-runner-version (a caller-controlled input) and then written unsanitized to $GITHUB_OUTPUT multiple times: e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A malicious caller could inject newlines into the input to poison subsequent GITHUB_OUTPUT entries.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is populated from steps.cache-paths.outputs.sbt_toolpath (a step output that itself contains the unsanitized inputs.sbt-runner-version value). The step executes `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` (or `$PWD\\sbt\\bin` on Windows) to $GITHUB_PATH without any sanitization. Because $PWD is derived from the attacker-controlled $SBT_TOOLPATH, this constitutes an indirect write of untrusted data to $GITHUB_PATH without the required `printf '%s' ... | tr -d '\n\r'` sanitization step.

Locations:

- `action.yml:133`
- `action.yml:135`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step (lines 27-42): Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block to strip newlines from the caller-controlled input before it is embedded in any $GITHUB_OUTPUT writes (sbt_toolpath, sbt_cachekey, etc.).
2. 'Setup PATH' step (lines 133-135): Added sanitization of SBT_TOOLPATH with `printf '%s' ... | tr -d '\n\r'` before using it, and sanitized the resulting $PWD-derived path strings before writing to $GITHUB_PATH in both the Windows and non-Windows branches.

