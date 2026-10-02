<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from inputs.sbt-runner-version (caller-controlled untrusted input) and is written unsanitized into $GITHUB_OUTPUT multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A newline embedded in the version input could inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:22`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is populated from steps.cache-paths.outputs.sbt_toolpath, which was itself constructed from the untrusted inputs.sbt-runner-version. After `cd "$SBT_TOOLPATH"`, the value `$PWD/sbt/bin` (tainted by the original input) is written unsanitized to $GITHUB_PATH via `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied, allowing a newline in the input to inject arbitrary entries into GITHUB_PATH.

Locations:

- `action.yml:196`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/action.yml:

1. 'Set up cache paths' step (line 22): Added sanitization of SBT_RUNNER_VERSION using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script. All GITHUB_OUTPUT writes that included the version now use `$SAFE_SBT_RUNNER_VERSION` instead of the raw `$SBT_RUNNER_VERSION`.

2. 'Setup PATH' step (line 196): Replaced direct `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` with a sanitized form: `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` followed by `echo "$safe_path" >> "$GITHUB_PATH"` (both for Linux/macOS and Windows paths). This prevents newline injection into GITHUB_PATH from the tainted path value derived from the untrusted sbt-runner-version input.

