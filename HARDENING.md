<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from the untrusted input `inputs.sbt-runner-version` via an env: block, then writes it directly into $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller-controlled newline in the version string could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps.

Locations:

- `action.yml:27`

### github-env-injection (severity: high)

The 'Setup PATH' step sets SBT_TOOLPATH from `steps.cache-paths.outputs.sbt_toolpath` (a workflow-controllable step output) via an env: block, then writes `$PWD/sbt/bin` to $GITHUB_PATH after `cd "$SBT_TOOLPATH"`. Because $PWD is derived from the untrusted SBT_TOOLPATH value, the path written to $GITHUB_PATH is attacker-influenced and is not sanitized with `printf '%s' ... | tr -d '\n\r'` before the write.

Locations:

- `action.yml:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (line 27): Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script to sanitize the caller-controlled input before it is embedded in any GITHUB_OUTPUT writes (sbt_toolpath, sbt_cachekey, etc.).

2. 'Setup PATH' step (line 155): Added `SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` to sanitize the step-output-derived value, then replaced the `$PWD`-based path construction with direct use of the sanitized `$SBT_TOOLPATH` variable, with an additional `printf '%s' ... | tr -d '\n\r'` sanitization before writing to GITHUB_PATH. This eliminates the attacker-influenced $PWD vector.

