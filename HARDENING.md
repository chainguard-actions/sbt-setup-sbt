<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `inputs.sbt-runner-version` (a caller-controlled input) and then written into `$GITHUB_OUTPUT` multiple times without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). Affected writes include: `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`, `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`, and `echo "sbt_diskcachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. A newline embedded in the version input could inject arbitrary key=value pairs into the output context.

Locations:

- `action.yml:24`

### github-env-injection (severity: high)

In the 'Setup PATH' step, `SBT_TOOLPATH` is set from `steps.cache-paths.outputs.sbt_toolpath`, which itself was built from the unsanitized `inputs.sbt-runner-version`. After `cd "$SBT_TOOLPATH"`, the resulting `$PWD` (which reflects the attacker-controlled path) is written directly to `$GITHUB_PATH` via `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` without any `tr -d '\n\r'` sanitization. A newline in the version input could inject an arbitrary path entry into GITHUB_PATH.

Locations:

- `action.yml:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (line 24): Added sanitization of SBT_RUNNER_VERSION at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All three GITHUB_OUTPUT writes that included the version string now use `$SAFE_SBT_RUNNER_VERSION` instead of `$SBT_RUNNER_VERSION`.

2. 'Setup PATH' step (line 155): Added sanitization of the path before writing to GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and the Windows equivalent), then writing `$safe_path` to `$GITHUB_PATH` instead of the raw `$PWD`-derived value.

