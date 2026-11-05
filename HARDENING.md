<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.14** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Set up cache paths' writes SBT_RUNNER_VERSION to $GITHUB_OUTPUT without the `printf '%s' ... | tr -d '\n\r'` cleaning step. That env var is set from ${{ inputs.sbt-runner-version }}. Affected lines: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (and the Windows variant), plus `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. If the input contains a newline, an attacker can add extra outputs or overwrite existing ones.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

Step 'Setup PATH' writes a path to $GITHUB_PATH without the `printf '%s' ... | tr -d '\n\r'` cleaning step: `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` (and the Windows variant). The path is built from SBT_TOOLPATH, which comes from ${{ steps.cache-paths.outputs.sbt_toolpath }}. That output was itself built from inputs.sbt-runner-version.

Locations:

- `action.yml:205`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Both findings are fixed in action.yml. I haven't run the action to test the changes.

- **'Set up cache paths' step:** the first line of the script now strips newlines from the sbt-runner-version input (`SBT_RUNNER_VERSION="$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')"`). Every output built from it (sbt_toolpath on all three runner OSes, and sbt_cachekey) now gets the cleaned value. Nothing else in the step changed.
- **'Setup PATH' step:** the bin path (Windows or Unix form) is first stored in a variable. It is then written with `printf '%s' "$SBT_BIN_PATH" | tr -d '\n\r' >> "$GITHUB_PATH"`, followed by `echo >> "$GITHUB_PATH"` so the entry still ends in a newline.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection findings in action.yml. I haven't run the action to test these changes.

- 'Set up cache paths': added a write_output helper that strips CR/LF from each value before writing key=value to GITHUB_OUTPUT. Every output now goes through it, including the ones built from HOME, LOCALAPPDATA and the RUNNER_* variables. The date result is captured first with '|| true'. Upstream ran date inside echo's arguments, so a failing date never stopped the step, and the '|| true' keeps it that way.
- 'Setup PATH': the sbt/bin path built from $PWD is now passed through printf '%s' | tr -d '\n\r' right before it is written to GITHUB_PATH, on both Windows and other runners.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

I fixed the github-env-injection finding in action.yml. I haven't run the action to test it.

In the 'Set up cache paths' step, each inherited variable now has newlines and carriage returns stripped (printf '%s' ... | tr -d '\n\r') right after the existing SBT_RUNNER_VERSION line. That covers RUNNER_TOOL_CACHE, RUNNER_TEMP, LOCALAPPDATA, HOME, RUNNER_OS and RUNNER_ARCH, which the finding listed. I also did the same for SBT_CACHE_KEY_VERSION and JDK_VERSION (the version read from `java -version`), since both are written to the outputs too.

The echo lines that write the outputs are unchanged, so a value with no newline produces exactly the same output as before. The two `sbt_monday` lines still pass `date`'s output straight into echo. I left them as upstream wrote them because `date` with a fixed format prints a single line.

LOCALAPPDATA is read as `${LOCALAPPDATA:-}`, since it is normally only set on Windows.

