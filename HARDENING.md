<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step writes the value of $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version, an untrusted caller-controlled input) directly into $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-supplied version string containing embedded newlines could inject arbitrary key=value pairs into $GITHUB_OUTPUT, poisoning subsequent steps. Affected lines include: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`, `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`, and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:21`

### github-env-injection (severity: high)

The 'Setup PATH' step writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization. $PWD is set by `cd "$SBT_TOOLPATH"`, where SBT_TOOLPATH comes from steps.cache-paths.outputs.sbt_toolpath — a step output that itself embeds the unsanitized inputs.sbt-runner-version value. A newline-containing version string could inject arbitrary entries into $GITHUB_PATH. The write `echo "$PWD\\sbt\\bin" >> "$GITHUB_PATH"` / `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` is not preceded by the required sanitization pipeline.

Locations:

- `action.yml:183`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step (line 21): Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and replaced all uses of $SBT_RUNNER_VERSION in $GITHUB_OUTPUT writes with the sanitized variable. This prevents newline injection via the caller-controlled sbt-runner-version input.
2. 'Setup PATH' step (line 183): Added `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows variant) before writing to $GITHUB_PATH, using the sanitized variable instead of the raw path. This prevents newline injection via the SBT_TOOLPATH value which embeds the unsanitized version string.

