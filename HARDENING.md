<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the env var SBT_RUNNER_VERSION is populated from the untrusted input `inputs.sbt-runner-version` (via `${{ inputs.sbt-runner-version }}`), and is then written directly to $GITHUB_OUTPUT in multiple echo statements without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller-controlled value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. Affected lines include the writes of sbt_toolpath, sbt_cachekey, and sbt_diskcachekey that embed $SBT_RUNNER_VERSION.

Locations:

- `action.yml:22`
- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`
- `action.yml:47`
- `action.yml:52`
- `action.yml:55`
- `action.yml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of SBT_RUNNER_VERSION at the start of the 'Set up cache paths' run script in action.yml. The fix uses `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the script, stripping newlines and carriage returns from the caller-controlled input before it is embedded in any echo statements that write to $GITHUB_OUTPUT (sbt_toolpath, sbt_cachekey, sbt_diskcachekey). This prevents newline injection attacks that could inject arbitrary key=value pairs into GITHUB_OUTPUT.

