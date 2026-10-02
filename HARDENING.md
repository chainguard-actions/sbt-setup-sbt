<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the caller-controlled input `inputs.sbt-runner-version` and then written unsanitized to $GITHUB_OUTPUT multiple times (e.g. `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. An attacker who controls the `sbt-runner-version` input can inject newline characters to smuggle arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps.

Locations:

- `action.yml:26`
- `action.yml:31`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the 'Set up cache paths' step's run script. This sanitizes the caller-controlled `inputs.sbt-runner-version` value (already placed in the env block as SBT_RUNNER_VERSION) by stripping all newline and carriage return characters before it is used in any writes to $GITHUB_OUTPUT (lines 26, 31, 37, 42). This prevents newline injection attacks that could smuggle arbitrary key=value pairs into GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml to sanitize path values before writing to $GITHUB_PATH. Both the Windows path ($PWD\\sbt\\bin) and Unix path ($PWD/sbt/bin) are now sanitized using `safe=$(printf '%s' "$PATH_VALUE" | tr -d '\n\r')` before being echoed to $GITHUB_PATH, preventing newline injection attacks via the SBT_TOOLPATH environment variable which is derived from the untrusted steps.cache-paths.outputs.sbt_toolpath value.

