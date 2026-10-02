<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.23** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the user-controlled input `inputs.sbt-runner-version` to the env var `SBT_RUNNER_VERSION` and then writes it directly into `$GITHUB_OUTPUT` on multiple lines without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who controls the `sbt-runner-version` input could inject a newline character into the value, causing arbitrary additional key=value pairs to be written to GITHUB_OUTPUT and potentially poisoning downstream steps. Affected lines:
- Line 19: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`
- Line 21: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`
- Line 24: `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`

Fix: sanitize the value before writing, e.g.:
```bash
safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')
echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$safe_version" >> "$GITHUB_OUTPUT"
```

Locations:

- `action.yml:19`
- `action.yml:21`
- `action.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, then replaced all three GITHUB_OUTPUT writes that used `$SBT_RUNNER_VERSION` to use `$safe_version` instead. This prevents newline injection via the user-controlled `sbt-runner-version` input from poisoning downstream steps.

