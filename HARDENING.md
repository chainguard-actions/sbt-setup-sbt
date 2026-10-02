<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from the untrusted input `inputs.sbt-runner-version` via env: and then writes it unsanitized to $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes, allowing a newline-injection attack to poison $GITHUB_OUTPUT with arbitrary key=value pairs.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:38`
- `action.yml:43`

### github-env-injection (severity: high)

The 'Setup PATH' step sets SBT_TOOLPATH from `steps.cache-paths.outputs.sbt_toolpath` (itself derived from the untrusted input `inputs.sbt-runner-version`) via env:, then executes `cd "$SBT_TOOLPATH"` and writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization (`echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). Because $PWD is set by cd-ing into an attacker-influenced path, the value written to $GITHUB_PATH is indirectly controlled by untrusted input and is not sanitized with `printf '%s' ... | tr -d '\n\r'` before the write.

Locations:

- `action.yml:208`
- `action.yml:210`

### script-injection (severity: high)

Rule (b) violation: The 'Download and Install sbt' step uses $SBT_RUNNER_VERSION (sourced from the untrusted input `inputs.sbt-runner-version` via env:) unquoted inside URL strings in curl commands: `curl -sL "https://github.com/sbt/sbt/releases/download/v$SBT_RUNNER_VERSION/sbt-$SBT_RUNNER_VERSION.zip" > ...`. While the variable is inside a double-quoted string, the URL is constructed by embedding the unquoted variable directly, and the variable itself is not double-quoted as a discrete token. An attacker-controlled version string containing shell metacharacters (e.g. `$(...)`, backticks) could break out of the string context if the quoting is not strictly maintained throughout. More critically, the same unquoted $SBT_RUNNER_VERSION is used in path arguments and filenames without isolation.

Locations:

- `action.yml:76`
- `action.yml:78`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:

1. github-env-injection (Set up cache paths): Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block and replaced all $SBT_RUNNER_VERSION references in $GITHUB_OUTPUT writes with the sanitized variable.

2. github-env-injection (Setup PATH): Added sanitization with `SAFE_PATH=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows equivalent) before writing to $GITHUB_PATH.

3. script-injection (Download and Install sbt): Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block and replaced all $SBT_RUNNER_VERSION references in curl URLs, output filenames, and unzip arguments with `${SAFE_SBT_RUNNER_VERSION}`.

