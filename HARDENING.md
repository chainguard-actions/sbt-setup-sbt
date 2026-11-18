<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.15** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Set up cache paths' writes the env var SBT_RUNNER_VERSION, which comes straight from inputs.sbt-runner-version, into $GITHUB_OUTPUT with no sanitization step (printf '%s' ... | tr -d '\n\r'). Examples: echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT" and echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT". If the input contains a newline, it can add arbitrary step outputs such as sbt_toolpath or sbt_downloadpath. Later steps use those outputs for paths and the PATH entry.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

Step 'Setup PATH' runs cd "$SBT_TOOLPATH", where SBT_TOOLPATH is steps.cache-paths.outputs.sbt_toolpath and is derived from the input. It then writes $PWD to $GITHUB_PATH (echo "$PWD/sbt/bin" >> "$GITHUB_PATH") with no printf '%s' ... | tr -d '\n\r' sanitization. A path containing a newline would add extra, attacker-chosen PATH entries.

Locations:

- `action.yml:240`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

I fixed both github-env-injection findings in action.yml by stripping CR/LF characters before anything is written to the runner files. I haven't run the action to test the change.

1. 'Set up cache paths' step: the first line now cleans SBT_RUNNER_VERSION with printf '%s' | tr -d '\n\r'. Every output written later in that step (sbt_toolpath on all three operating systems, and sbt_cachekey) uses the cleaned value. A version string with no newline produces the same outputs as before.

2. 'Setup PATH' step: $PWD is cleaned the same way into SAFE_PWD, and that is what gets written to $GITHUB_PATH. The Windows backslash path and the Unix path are both unchanged otherwise.

The 'Download and Install sbt' step still reads the raw input in its download URLs. That step writes nothing to GITHUB_OUTPUT, GITHUB_ENV or GITHUB_PATH, so it was not part of these findings and I left it alone.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set up cache paths' step in action.yml. Each variable it reads from the environment is now cleaned first with printf '%s' | tr -d '\n\r', into its own SAFE_* copy: RUNNER_TOOL_CACHE, RUNNER_TEMP, LOCALAPPDATA, HOME, RUNNER_OS and RUNNER_ARCH. I did the same for SBT_CACHE_KEY_VERSION and for JDK_VERSION, which comes from java output. Every write to $GITHUB_OUTPUT and every OS check now uses the cleaned copies. The output names and the values they produce are unchanged for normal input. The original variables stay as they are, so later commands still see them unchanged. I haven't run the action or tested this change.

