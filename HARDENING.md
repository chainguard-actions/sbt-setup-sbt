<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets the env var SBT_RUNNER_VERSION from the untrusted input `inputs.sbt-runner-version` and then writes it unsanitized into $GITHUB_OUTPUT multiple times — e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. An attacker-controlled value containing newlines could inject arbitrary key=value pairs into the output context. The required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent before every such write.

Locations:

- `action.yml:20`

### github-env-injection (severity: high)

The 'Setup PATH' step sets SBT_TOOLPATH from `steps.cache-paths.outputs.sbt_toolpath` (which was itself derived from the untrusted `inputs.sbt-runner-version`). The step does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to `$GITHUB_PATH` without sanitization. Because $PWD reflects the attacker-controlled path, the value appended to $GITHUB_PATH is transitively tainted. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent before the write.

Locations:

- `action.yml:228`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (line 20): Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script to sanitize the untrusted `inputs.sbt-runner-version` value before it is embedded in any `echo ... >> "$GITHUB_OUTPUT"` statements.

2. 'Setup PATH' step (line 228): Added `SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` to sanitize the transitively tainted path, then replaced the `$PWD`-based writes with explicit sanitized path construction (`safe_path=$(printf '%s' "$SBT_TOOLPATH/sbt/bin" | tr -d '\n\r')`) before writing to `$GITHUB_PATH`. This eliminates the newline injection risk in both GITHUB_OUTPUT and GITHUB_PATH writes.

