<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from inputs.sbt-runner-version and then written directly into $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller-controlled newline in the input could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:22`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is populated from steps.cache-paths.outputs.sbt_toolpath (which itself embeds the caller-controlled inputs.sbt-runner-version value) and then written directly into $GITHUB_PATH (e.g., `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`) without the required sanitization step. A newline embedded in the path value could inject an arbitrary additional entry into $GITHUB_PATH.

Locations:

- `action.yml:196`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (line 22): Added sanitization `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and replaced all uses of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT writes with `$SAFE_SBT_RUNNER_VERSION` to prevent newline injection.

2. 'Setup PATH' step (line 196): Added sanitization `SAFE_SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` and replaced `cd "$SBT_TOOLPATH"` with `cd "$SAFE_SBT_TOOLPATH"` so that `$PWD` (used in the GITHUB_PATH write) is derived from the sanitized path, preventing newline injection into GITHUB_PATH.

