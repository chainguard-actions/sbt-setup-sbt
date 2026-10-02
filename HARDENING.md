<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the user-controlled input `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION`, then writes it to `$GITHUB_OUTPUT` multiple times without sanitization. For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. A calling workflow can supply a version string containing newlines, which would allow injecting arbitrary key=value pairs into GITHUB_OUTPUT (and potentially GITHUB_ENV/GITHUB_PATH in downstream steps that consume these outputs). The required sanitization step — `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` — is absent before every write.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in the 'Set up cache paths' step of action.yml. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the beginning of the run script to sanitize the user-controlled `inputs.sbt-runner-version` input. Replaced all uses of `$SBT_RUNNER_VERSION` in echo commands writing to $GITHUB_OUTPUT with `$SAFE_SBT_RUNNER_VERSION`. This prevents an attacker from injecting arbitrary key=value pairs into GITHUB_OUTPUT by supplying a version string containing newline characters.

