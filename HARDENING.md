<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the untrusted input `inputs.sbt-runner-version` (via `env: SBT_RUNNER_VERSION: ${{ inputs.sbt-runner-version }}`), and is then written directly into $GITHUB_OUTPUT multiple times without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A caller-controlled newline in the version string could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step, added newline sanitization for the SBT_RUNNER_VERSION input before it is written to $GITHUB_OUTPUT. A new variable SAFE_SBT_RUNNER_VERSION is computed at the top of the run script using `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`, and all occurrences of $SBT_RUNNER_VERSION in $GITHUB_OUTPUT writes (sbt_toolpath and sbt_cachekey) are replaced with $SAFE_SBT_RUNNER_VERSION. This prevents a caller-controlled newline in the version string from injecting arbitrary key=value pairs into GITHUB_OUTPUT.

