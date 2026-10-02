<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the env var SBT_RUNNER_VERSION is populated from the user-controlled input `inputs.sbt-runner-version` (via `env: SBT_RUNNER_VERSION: ${{ inputs.sbt-runner-version }}`), and is then written directly into $GITHUB_OUTPUT multiple times without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). For example:
  echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"
  echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"
An attacker who controls the `sbt-runner-version` input could inject newline characters to add arbitrary key=value pairs into GITHUB_OUTPUT, potentially hijacking the values read by downstream steps (e.g. sbt_toolpath, sbt_downloadpath, sbt_diskcache). The fix is to sanitize the value before writing: `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and use `$safe` in the echo statements.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added sanitization of the user-controlled SBT_RUNNER_VERSION input before writing to $GITHUB_OUTPUT. Added `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, and replaced all uses of `$SBT_RUNNER_VERSION` in echo-to-GITHUB_OUTPUT statements with `$safe_version`. This prevents newline injection attacks that could allow an attacker to inject arbitrary key=value pairs into GITHUB_OUTPUT.

