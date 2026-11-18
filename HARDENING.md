<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.13** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Set up cache paths' writes values built from SBT_RUNNER_VERSION to $GITHUB_OUTPUT without first stripping newlines. SBT_RUNNER_VERSION comes from env: ${{ inputs.sbt-runner-version }}. Offending lines include: echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT" (in the Windows, macOS and Linux branches) and echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT". An input value containing a newline could inject extra output keys. Fix: set safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r') and use $safe in these writes.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

Step 'Setup PATH' runs cd "$SBT_TOOLPATH", where SBT_TOOLPATH comes from ${{ steps.cache-paths.outputs.sbt_toolpath }}, a step output partly built from inputs.sbt-runner-version. It then writes "$PWD/sbt/bin" (and on Windows "$PWD\\sbt\\bin") to $GITHUB_PATH without the printf '%s' ... | tr -d '\n\r' step. $GITHUB_PATH has no heredoc form, so the newline strip is required.

Locations:

- `action.yml:214`
- `action.yml:216`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index d3fa1e3..3f8f727 100644
--- a/action.yml
+++ b/action.yml
@@ -12,6 +12,23 @@ inputs:
 runs:
   using: "composite"
   steps:
     - name: Set up cache paths
       id: cache-paths
       shell: bash
@@ -23,24 +40,33 @@ runs:
         if [[ -x "$(command -v java)" ]]; then
           JDK_VERSION=$(java -version 2>&1 | head -n 1 | sed -e 's/.*"\(.*\)"\(.*\)/\1/; 1q')
         fi
+        safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')
+        safe_jdk=$(printf '%s' "$JDK_VERSION" | tr -d '\n\r')
+        safe_tool_cache=$(printf '%s' "$RUNNER_TOOL_CACHE" | tr -d '\n\r')
+        safe_temp=$(printf '%s' "$RUNNER_TEMP" | tr -d '\n\r')
+        safe_home=$(printf '%s' "$HOME" | tr -d '\n\r')
         if [[ "$RUNNER_OS" == "Windows" ]]; then
-          echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"
-          echo "sbt_downloadpath=$RUNNER_TEMP\\_sbt" >> "$GITHUB_OUTPUT"
-          echo "sbt_diskcache=$LOCALAPPDATA\\sbt" >> "$GITHUB_OUTPUT"
+          safe_localappdata=$(printf '%s' "$LOCALAPPDATA" | tr -d '\n\r')
+          echo "sbt_toolpath=$safe_tool_cache\\sbt\\$safe_version" >> "$GITHUB_OUTPUT"
+          echo "sbt_downloadpath=$safe_temp\\_sbt" >> "$GITHUB_OUTPUT"
+          echo "sbt_diskcache=$safe_localappdata\\sbt" >> "$GITHUB_OUTPUT"
           echo "sbt_monday=$(date -d "last Monday" +%Y-%m-%d)" >> "$GITHUB_OUTPUT"
         elif [[ "$RUNNER_OS" == "macOS" ]]; then
-          echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"
-          echo "sbt_downloadpath=$RUNNER_TEMP/_sbt" >> "$GITHUB_OUTPUT"
-          echo "sbt_diskcache=$HOME/Library/Caches/sbt" >> "$GITHUB_OUTPUT"
+          echo "sbt_toolpath=$safe_tool_cache/sbt/$safe_version" >> "$GITHUB_OUTPUT"
+          echo "sbt_downloadpath=$safe_temp/_sbt" >> "$GITHUB_OUTPUT"
+          echo "sbt_diskcache=$safe_home/Library/Caches/sbt" >> "$GITHUB_OUTPUT"
           echo "sbt_monday=$(date -v-monday -v+0d +%Y-%m-%d)" >> "$GITHUB_OUTPUT"
         else
-          echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"
-          echo "sbt_downloadpath=$RUNNER_TEMP/_sbt" >> "$GITHUB_OUTPUT"
-          echo "sbt_diskcache=$HOME/.cache/sbt" >> "$GITHUB_OUTPUT"
+          echo "sbt_toolpath=$safe_tool_cache/sbt/$safe_version" >> "$GITHUB_OUTPUT"
+          echo "sbt_downloadpath=$safe_temp/_sbt" >> "$GITHUB_OUTPUT"
+          echo "sbt_diskcache=$safe_home/.cache/sbt" >> "$GITHUB_OUTPUT"
           echo "sbt_monday=$(date -d "last Monday" +%Y-%m-%d)" >> "$GITHUB_OUTPUT"
         fi
-        echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"
-        echo "sbt_diskcachekey=$RUNNER_OS-$RUNNER_ARCH-java$JDK_VERSION-sbt-diskcache-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"
+        safe_os=$(printf '%s' "$RUNNER_OS" | tr -d '\n\r')
+        safe_arch=$(printf '%s' "$RUNNER_ARCH" | tr -d '\n\r')
+        safe_keyver=$(printf '%s' "$SBT_CACHE_KEY_VERSION" | tr -d '\n\r')
+        echo "sbt_cachekey=$safe_os-$safe_arch-sbt-runner-$safe_version-$safe_keyver" >> "$GITHUB_OUTPUT"
+        echo "sbt_diskcachekey=$safe_os-$safe_arch-java$safe_jdk-sbt-diskcache-$safe_keyver" >> "$GITHUB_OUTPUT"
     - name: Check tool cache
       id: cache-tool-dir
       shell: bash
@@ -56,7 +82,7 @@ runs:
     - name: Cache sbt distribution
       id: cache-dir
       if: steps.cache-tool-dir.outputs.cache-hit != 'true'
       uses: actions/cache@55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6.1.0
       with:
         path: ${{ steps.cache-paths.outputs.sbt_toolpath }}
         key: ${{ steps.cache-paths.outputs.sbt_cachekey }}
@@ -75,7 +101,7 @@ runs:
     - name: sbt 2.x disk cache
       id: disk-cache
       if: inputs.disk-cache == 'true'
       uses: actions/cache@55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6.1.0
       with:
         path: ${{ steps.cache-paths.outputs.sbt_diskcache }}
         key: ${{ steps.cache-paths.outputs.sbt_diskcachekey }}-${{ steps.cache-paths
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, prior-fix

**Notes:**

Both fixes are in action.yml.

1. 'Set up cache paths' step: I re-applied the previous harden's patch. Newlines and carriage returns are now stripped from the runner version input and from every other value (JDK version, tool cache, temp, HOME, LOCALAPPDATA, OS, arch, cache-key version) before it is written to $GITHUB_OUTPUT. This covers the Windows, macOS and Linux branches and both cache keys.

2. 'Setup PATH' step: $PWD is also stripped the same way before it is written to $GITHUB_PATH. $GITHUB_PATH has no multi-line (heredoc) form, so stripping is the only fix there.

The new safe_* variables only hold the stripped copies. Every path and cache key comes out the same as before for normal input. Nothing was run or tested; the change is a code edit only.

