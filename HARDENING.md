<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Set up cache paths': The env var SBT_RUNNER_VERSION is set from the untrusted input `${{ inputs.sbt-runner-version }}` and then written directly to $GITHUB_OUTPUT in multiple echo statements without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps. Affected lines include the Windows branch (line 27), macOS branch (line 32), Linux branch (line 37), and the sbt_cachekey echo (line 42).

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

Step 'Setup PATH': The env var SBT_TOOLPATH is set from `${{ steps.cache-paths.outputs.sbt_toolpath }}`, which is itself derived from the untrusted input `inputs.sbt-runner-version`. The script does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization. A newline-containing input value could inject additional entries into GITHUB_PATH, allowing path hijacking for subsequent steps.

Locations:

- `action.yml:222`
- `action.yml:224`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Sanitized SBT_RUNNER_VERSION (from untrusted inputs.sbt-runner-version) using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before embedding it in all four echo statements that write to $GITHUB_OUTPUT (Windows toolpath, macOS toolpath, Linux toolpath, and sbt_cachekey).
2. 'Setup PATH' step: Sanitized the path written to $GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` for both Windows and non-Windows branches, preventing newline injection into GITHUB_PATH.

