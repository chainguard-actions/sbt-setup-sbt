<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.13** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step writes SBT_RUNNER_VERSION, which comes from ${{ inputs.sbt-runner-version }}, to $GITHUB_OUTPUT with no sanitization. Examples: echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT" and echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT". A newline in the input could inject arbitrary step outputs. Inherited env vars such as $LOCALAPPDATA and $HOME are also written without sanitization. Fix: run safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r') first and write $safe instead.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:41`

### github-env-injection (severity: high)

The 'Setup PATH' step runs cd "$SBT_TOOLPATH". SBT_TOOLPATH comes from steps.cache-paths.outputs.sbt_toolpath, which is built from the input. The step then writes $PWD to $GITHUB_PATH (echo "$PWD/sbt/bin" >> "$GITHUB_PATH") with no printf '%s' ... | tr -d '\n\r' sanitization. Line number is approximate; the step is the last one in action.yml.

Locations:

- `action.yml:210`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection findings in action.yml. Changes were checked by reading the file only. I didn't run anything.

1. 'Set up cache paths' step: the `sbt-runner-version` input is now copied with newlines and carriage returns removed (`printf '%s' | tr -d '\n\r'`), and that copy is what goes into the `sbt_toolpath` and `sbt_cachekey` outputs. I did the same for the other values this step writes as outputs: `$RUNNER_TOOL_CACHE`, `$RUNNER_TEMP`, `$HOME`, `$LOCALAPPDATA` (Windows only) and the detected JDK version. The finding also named `$HOME` and `$LOCALAPPDATA`.

2. 'Setup PATH' step: `$PWD` gets the same newline removal before it is written to `$GITHUB_PATH`.

No other logic changed. For any normal value without newlines, the outputs and PATH entries come out exactly as before.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step, RUNNER_OS, RUNNER_ARCH and SBT_CACHE_KEY_VERSION now go through printf '%s' | tr -d '\n\r' (stored as safe_os, safe_arch, safe_keyver) before they are written to the sbt_cachekey and sbt_diskcachekey outputs. The output values stay the same for normal inputs.

