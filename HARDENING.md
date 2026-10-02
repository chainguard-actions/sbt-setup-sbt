<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from inputs.sbt-runner-version via the env: block, then writes it unsanitized to $GITHUB_OUTPUT in multiple echo statements (e.g. `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller-controlled value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps.

Locations:

- `action.yml:26`
- `action.yml:30`
- `action.yml:35`
- `action.yml:40`

### script-injection (severity: high)

Rule (b) violation: The 'Download and Install sbt' step uses $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version via env:) unquoted inside curl URL arguments: `curl -sL "https://github.com/sbt/sbt/releases/download/v$SBT_RUNNER_VERSION/sbt-$SBT_RUNNER_VERSION.zip"`. Although the outer string is double-quoted, the variable is not separately guarded, allowing shell metacharacter injection from a workflow-controllable input value. Similarly, `unzip -o "sbt-$SBT_RUNNER_VERSION.zip"` uses the variable unquoted.

Locations:

- `action.yml:68`
- `action.yml:70`
- `action.yml:73`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed github-env-injection in 'Set up cache paths' step by sanitizing SBT_RUNNER_VERSION with `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` before all GITHUB_OUTPUT writes. Fixed script-injection in 'Download and Install sbt' step by sanitizing and strictly validating SBT_RUNNER_VERSION (must match ^[0-9]+\.[0-9]+\.[0-9]+$) before using it in curl URLs and unzip commands.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml (around line 207). The step was writing $PWD/sbt/bin directly to $GITHUB_PATH after cd-ing into $SBT_TOOLPATH (which is sourced from steps.cache-paths.outputs.sbt_toolpath). The fix adds sanitization using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_PATH for both Windows and non-Windows paths, preventing potential newline injection attacks.

