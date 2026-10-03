<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets the env var SBT_RUNNER_VERSION from the untrusted user-controlled input `${{ inputs.sbt-runner-version }}` and then writes it directly to $GITHUB_OUTPUT multiple times without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-supplied version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. Affected lines include writes such as: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added a sanitization line at the start of the run script: `SAFE_SBT_RUNNER_VERSION="$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')"`. All subsequent uses of the version string in $GITHUB_OUTPUT writes now reference $SAFE_SBT_RUNNER_VERSION instead of $SBT_RUNNER_VERSION, preventing an attacker from injecting arbitrary key=value pairs via newlines in the sbt-runner-version input.

