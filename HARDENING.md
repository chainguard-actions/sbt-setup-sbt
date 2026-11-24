<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.16

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.16** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Set up cache paths' (id: cache-paths) writes SBT_RUNNER_VERSION, which comes from inputs.sbt-runner-version, to $GITHUB_OUTPUT with no newline sanitization. Examples: echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT" and echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT". The same step also writes inherited env vars ($LOCALAPPDATA, $HOME) the same way. If the value contains a newline, extra output keys can be injected (for example a different sbt_toolpath, which later steps add to PATH). Fix: run safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r') before the writes and use $safe.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

Step 'Setup PATH' runs cd "$SBT_TOOLPATH" and then writes $PWD to $GITHUB_PATH (echo "$PWD/sbt/bin" >> "$GITHUB_PATH"). SBT_TOOLPATH comes from steps.cache-paths.outputs.sbt_toolpath, which is built from inputs.sbt-runner-version, so $PWD is workflow-controlled. A newline in it would add extra PATH entries. Nothing strips newlines before the write, and $GITHUB_PATH has no heredoc form. Fix: run safe=$(printf '%s' "$PWD" | tr -d '\n\r') and write "$safe/sbt/bin".

Locations:

- `action.yml:222`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

I fixed both github-env-injection findings in action.yml. Nothing was run or tested.

1. 'Set up cache paths' step: newlines and carriage returns are now stripped from SBT_RUNNER_VERSION, HOME, LOCALAPPDATA and JDK_VERSION before any of them is written to $GITHUB_OUTPUT. The sbt_toolpath, sbt_diskcache, sbt_cachekey and sbt_diskcachekey writes now use the cleaned values. JDK_VERSION wasn't in the finding; I cleaned it too because it is written to $GITHUB_OUTPUT the same way.

2. 'Setup PATH' step: newlines are stripped from $PWD before "$safe/sbt/bin" (or the Windows form) is written to $GITHUB_PATH.

The download and signature-check steps still read inputs.sbt-runner-version directly. That is unchanged from upstream: the finding only covered the writes to $GITHUB_OUTPUT and $GITHUB_PATH.

