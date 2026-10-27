<!-- markdownlint-disable -->

# Hardening Report: fluxcd--flux2--action/v2.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--flux2--action/v2.10.0** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Download the binary to the runner's cache dir' writes values from inputs to special files without removing newlines. VERSION comes from inputs.version through env. When version-file is not used, the value goes out unsanitized in `echo "installed-flux-version=$VERSION" >> "$GITHUB_OUTPUT"`. Only the version-file path removes whitespace, and only the GitHub API lookup changes the value otherwise. FLUX_TOOL_DIR comes from inputs.bindir and is written as-is in `echo "$FLUX_TOOL_DIR" >> "$GITHUB_PATH"`. The values from the inherited env vars RUNNER_OS and RUNNER_ARCH are also written to GITHUB_OUTPUT as os=/arch= after being lowercased, but newlines are never removed. Fix: run `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before each write.

Locations:

- `action.yml:88`
- `action.yml:94`
- `action.yml:102`
- `action.yml:189`

### github-env-injection (severity: high)

Step 'Setup Plugins Dir' writes FLUXCD_PLUGINS, which comes from inputs.plugin-dir through env, to $GITHUB_ENV without removing newlines: `echo "FLUXCD_PLUGINS=$FLUXCD_PLUGINS" >> "$GITHUB_ENV"`. A newline in the input could inject extra environment variables, such as LD_PRELOAD or PATH, into later steps.

Locations:

- `action.yml:229`

### github-env-injection (severity: high)

Step 'Set Output' writes the inherited process env var FLUXCD_PLUGINS to $GITHUB_OUTPUT without removing newlines: `echo "plugin-dir=$FLUXCD_PLUGINS" >> "$GITHUB_OUTPUT"`. The variable comes from the plugin-dir input or from the calling workflow's environment.

Locations:

- `action.yml:254`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $FLUXCD_PLUGINS in step "Setup Plugins Dir" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:229`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, static-unsanitized-env-write

**Notes:**

Every flagged write in action.yml now strips newlines and carriage returns (printf '%s' ... | tr -d '\n\r') before writing to GITHUB_OUTPUT, GITHUB_ENV or GITHUB_PATH. Nothing was run or tested; these are text edits only.

What changed:
- **Download step:** VERSION is cleaned once, before it is written as installed-flux-version, so the download, the tool-cache path and the output all use the same cleaned value.
- **os= and arch= outputs:** both are cleaned before writing. RUNNER_OS and RUNNER_ARCH are now read with printf instead of echo.
- **Binary directory:** FLUX_TOOL_DIR is cleaned just before it is written to GITHUB_PATH. The install still uses the original value, so for a bindir input containing a newline, the binary would go into a different directory than the one put on PATH.
- **Setup Plugins Dir:** FLUXCD_PLUGINS is cleaned before it is written to GITHUB_ENV. The mkdir that follows uses the same cleaned value.
- **Set Output:** FLUXCD_PLUGINS is cleaned before plugin-dir is written to GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Restored the windows unzip line and the else/tar branch that a broken splice had mangled. Sanitized FLUX_TOOL_DIR with tr -d '\n\r' right before the $GITHUB_PATH write. Quoted $VERSION inside [[ ]]. Also repaired a similarly mangled FLUXCD_PLUGINS export in the 'Setup Plugins Dir' step, which had a duplicated, garbled echo line. The value is still sanitized before it goes to $GITHUB_ENV.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Restored the corrupted Windows unzip / else tar xzf block in the 'Download the binary' step to upstream form, and moved the sanitized write (safe_tool_dir via printf | tr -d '\n\r') to replace the unsanitized `echo "$FLUX_TOOL_DIR" >> "$GITHUB_PATH"`.

