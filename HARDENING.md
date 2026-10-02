<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.23** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from inputs.sbt-runner-version (an untrusted caller-controlled input) and then written unsanitized into $GITHUB_OUTPUT multiple times: `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before these writes. A crafted version string containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:19`
- `action.yml:21`
- `action.yml:24`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is populated from steps.cache-paths.outputs.sbt_toolpath, which itself contains the unsanitized inputs.sbt-runner-version value. The step does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization. A crafted version string with embedded newlines could inject arbitrary entries into $GITHUB_PATH.

Locations:

- `action.yml:237`
- `action.yml:239`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (lines 19-24): Added `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` to strip newlines from the caller-controlled `inputs.sbt-runner-version` value before writing it into `$GITHUB_OUTPUT` via the `sbt_toolpath` and `sbt_cachekey` echo commands.

2. 'Setup PATH' step (lines 237-239): Added `safe_pwd=$(printf '%s' "$PWD" | tr -d '\n\r')` to strip newlines from the working directory path (which is derived from the unsanitized version string) before writing it to `$GITHUB_PATH`.

Both fixes use the standard `printf '%s' ... | tr -d '\n\r'` sanitization pattern to prevent newline injection attacks.

