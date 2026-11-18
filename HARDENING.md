<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.14** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Set up cache paths' writes the env var SBT_RUNNER_VERSION, which comes from inputs.sbt-runner-version, into $GITHUB_OUTPUT with no `printf '%s' ... | tr -d '\n\r'` sanitization. Examples: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. If the input contains a newline, an attacker can add extra output entries (for example, overwrite sbt_toolpath or sbt_downloadpath). The same lines also forward inherited env vars (RUNNER_TOOL_CACHE, RUNNER_TEMP, LOCALAPPDATA, HOME) unsanitized.

Locations:

- `action.yml:27`

### github-env-injection (severity: high)

Step 'Setup PATH' does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` (or `$PWD\\sbt\\bin` on Windows) to $GITHUB_PATH with no `printf '%s' ... | tr -d '\n\r'` sanitization. SBT_TOOLPATH comes from steps.cache-paths.outputs.sbt_toolpath, which is built from inputs.sbt-runner-version, so the PATH entry is controlled by the workflow.

Locations:

- `action.yml:208`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index c741974..d229165 100644
--- a/action.yml
+++ b/action.yml
@@ -12,6 +12,23 @@ inputs:
 runs:
   using: "composite"
   steps:
     - name: Set up cache paths
       id: cache-paths
       shell: bash
@@ -19,10 +36,19 @@ runs:
         SBT_RUNNER_VERSION: ${{ inputs.sbt-runner-version }}
         SBT_CACHE_KEY_VERSION: 1.5.3
       run: |
+        SBT_RUNNER_VERSION="$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')"
+        RUNNER_TOOL_CACHE="$(printf '%s' "$RUNNER_TOOL_CACHE" | tr -d '\n\r')"
+        RUNNER_TEMP="$(printf '%s' "$RUNNER_TEMP" | tr -d '\n\r')"
+        LOCALAPPDATA="$(printf '%s' "${LOCALAPPDATA:-}" | tr -d '\n\r')"
+        HOME="$(printf '%s' "$HOME" | tr -d '\n\r')"
+        RUNNER_OS="$(printf '%s' "$RUNNER_OS" | tr -d '\n\r')"
+        RUNNER_ARCH="$(printf '%s' "$RUNNER_ARCH" | tr -d '\n\r')"
+        SBT_CACHE_KEY_VERSION="$(printf '%s' "$SBT_CACHE_KEY_VERSION" | tr -d '\n\r')"
         JDK_VERSION=0
         if [[ -x "$(command -v java)" ]]; then
           JDK_VERSION=$(java -version 2>&1 | head -n 1 | sed -e 's/.*"\(.*\)"\(.*\)/\1/; 1q')
         fi
+        JDK_VERSION="$(printf '%s' "$JDK_VERSION" | tr -d '\n\r')"
         if [[ "$RUNNER_OS" == "Windows" ]]; then
           echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"
           echo "sbt_downloadpath=$RUNNER_TEMP\\_sbt" >> "$GITHUB_OUTPUT"
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
         key: ${{ steps.cache-paths.outputs.sbt_diskcachekey }}-${{ steps.cache-paths.outputs.sbt_monday }}-${{ hashFiles('**/*.sbt', '**/*.properties') }}
@@ -210,7 +236,9 @@ runs:
         cd "$SBT_TOOLPATH"
         ls sbt/bin/sbt
         if [[ "$RUNNER_OS" == "Windows" ]]; then
-          echo "$PWD\\sbt\\bin" >> "$GITHUB_PATH"
+          SBT_BIN="$PWD\\sbt\\bin"
         else
-          echo "$PWD/sbt/bin" >> "$GITHUB_PATH"
+          SBT_BIN="$PWD/sbt/bin"
         fi
+        SAFE_SBT_BIN="$(printf '%s' "$SBT_BIN" | tr -d '\n\r')"
+        printf '%s\n' "$SAFE_SBT_BIN" >> "$GITHUB_PATH"

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, prior-fix

**Notes:**

Re-applied the previous harden's changes to action.yml. In 'Set up cache paths', newlines and carriage returns are now stripped from SBT_RUNNER_VERSION, RUNNER_TOOL_CACHE, RUNNER_TEMP, LOCALAPPDATA, HOME, RUNNER_OS, RUNNER_ARCH, SBT_CACHE_KEY_VERSION and JDK_VERSION with printf '%s' | tr -d '\n\r' before anything is written to $GITHUB_OUTPUT. In 'Setup PATH', the sbt bin path for Windows and Unix is built into a variable, stripped the same way, and written to $GITHUB_PATH with printf. The Windows backslash handling works as before. The actions/cache and carabiner-dev uses: lines were already pinned to commit SHAs, so I left them alone. Nothing has been run to test these changes.

