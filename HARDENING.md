<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.7** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets `SBT_RUNNER_VERSION` from `inputs.sbt-runner-version` via the `env:` block and then writes it unsanitized into `$GITHUB_OUTPUT` multiple times — e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. Because `inputs.sbt-runner-version` is caller-controlled, a newline embedded in the value would allow injecting arbitrary key=value pairs into the output file. The required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent before every write. This is a case-(d) violation (indirect write of inputs via env var without sanitization).

Locations:

- `action.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the run script to sanitize the caller-controlled `inputs.sbt-runner-version` value before it is written to $GITHUB_OUTPUT multiple times (in sbt_toolpath, sbt_cachekey, and other output values). This prevents newline injection attacks where an attacker could embed newlines in the version input to inject arbitrary key=value pairs into the GitHub output file.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in hardened/action/action.yml. The path written to $GITHUB_PATH is now sanitized using `printf '%s' ... | tr -d '\n\r'` before being written, preventing newline injection via the untrusted `steps.cache-paths.outputs.sbt_toolpath` value. Both Windows and non-Windows branches are sanitized.

