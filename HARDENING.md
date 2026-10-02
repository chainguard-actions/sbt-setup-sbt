<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the caller-controlled input `inputs.sbt-runner-version` and then written directly into $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`, `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`, `echo "sbt_diskcachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller supplying a version string containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning the outputs consumed by downstream steps in the same job.

Locations:

- `action.yml:19`
- `action.yml:27`
- `action.yml:31`
- `action.yml:36`
- `action.yml:41`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the run script. This sanitizes the caller-controlled `inputs.sbt-runner-version` value (already placed in the env block) by stripping embedded newlines and carriage returns before it is used in any `$GITHUB_OUTPUT` writes (sbt_toolpath, sbt_cachekey, sbt_diskcachekey). This prevents newline injection attacks that could poison downstream step outputs.

