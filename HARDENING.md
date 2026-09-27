<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from the untrusted input `inputs.sbt-runner-version` via env:, then writes values containing $SBT_RUNNER_VERSION directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` applied). An attacker-controlled version string containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. Affected lines include all echo statements writing sbt_toolpath, sbt_downloadpath, sbt_diskcache, sbt_cachekey, and sbt_diskcachekey.

Locations:

- `action.yml:23`
- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:36`

### github-env-injection (severity: high)

The 'Check Tool Cache' step sets SBT_TOOLPATH from `steps.cache-paths.outputs.sbt_toolpath` (itself derived from the untrusted `inputs.sbt-runner-version`) via env:, then writes 'cache-hit=true' and 'cache-hit=false' to $GITHUB_OUTPUT using $SBT_TOOLPATH in the conditional path check. More critically, the unsanitized $SBT_TOOLPATH value is used in `mkdir -p "$SBT_TOOLPATH"` without sanitization before the GITHUB_OUTPUT write. The step output values are written without the required `printf '%s' ... | tr -d '\n\r'` sanitization.

Locations:

- `action.yml:44`
- `action.yml:47`

### github-env-injection (severity: high)

The 'Setup PATH' step sets SBT_TOOLPATH from `steps.cache-paths.outputs.sbt_toolpath` (derived from untrusted `inputs.sbt-runner-version`) via env:, then does `cd "$SBT_TOOLPATH"` and writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization. Since $SBT_TOOLPATH is attacker-influenced, a newline-containing value could inject additional entries into $GITHUB_PATH, allowing an attacker to prepend arbitrary directories to the runner's PATH. The required sanitization step (`printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r'`) is absent before the write.

Locations:

- `action.yml:23`
- `action.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed all three github-env-injection findings in action.yml:

1. 'Set up cache paths' step: Sanitized SBT_RUNNER_VERSION with `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before using it in all GITHUB_OUTPUT writes (sbt_toolpath, sbt_cachekey).

2. 'Check Tool Cache' step: Sanitized SBT_TOOLPATH with `safe_toolpath=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` before using it in the filesystem check, mkdir, and GITHUB_OUTPUT writes.

3. 'Setup PATH' step: Sanitized SBT_TOOLPATH before the `cd` command, and also sanitized `$PWD` with `safe_pwd=$(printf '%s' "$PWD" | tr -d '\n\r')` before writing to GITHUB_PATH in both Windows and non-Windows branches. This prevents newline injection attacks from attacker-controlled version strings.

