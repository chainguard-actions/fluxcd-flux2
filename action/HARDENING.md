<!-- markdownlint-disable -->

# Hardening Report: fluxcd--flux2--action/v2.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--flux2--action/v2.10.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Download the binary to the runner's cache dir' writes values from untrusted sources to special env files without the `printf '%s' ... | tr -d '\n\r'` sanitization step. (1) `echo "installed-flux-version=$VERSION" >> "$GITHUB_OUTPUT"`: VERSION comes from inputs.version, or from the file named by inputs.version-file. (2) `echo "$FLUX_TOOL_DIR" >> "$GITHUB_PATH"`: FLUX_TOOL_DIR comes from inputs.bindir. (3) os= and arch= are written to GITHUB_OUTPUT from the inherited RUNNER_OS/RUNNER_ARCH env vars, and only case conversion is applied first.

Locations:

- `action.yml:88`
- `action.yml:94`
- `action.yml:102`
- `action.yml:189`

### github-env-injection (severity: high)

Step 'Setup Plugins Dir' runs `echo "FLUXCD_PLUGINS=$FLUXCD_PLUGINS" >> "$GITHUB_ENV"`. FLUXCD_PLUGINS comes from inputs.plugin-dir, or from the step outputs installed-flux-version/os/arch, and is written without the `printf '%s' | tr -d '\n\r'` sanitization step. A newline in the value could inject extra environment variables.

Locations:

- `action.yml:229`

### github-env-injection (severity: high)

Step 'Set Output' runs `echo "plugin-dir=$FLUXCD_PLUGINS" >> "$GITHUB_OUTPUT"`. FLUXCD_PLUGINS is an inherited process env var (set through GITHUB_ENV from inputs.plugin-dir, and also settable by the calling workflow). It is forwarded without sanitization.

Locations:

- `action.yml:254`

### script-injection (severity: high)

Rule (b) violation in step 'Download the binary to the runner's cache dir': `if [[ $VERSION = v* ]]; then`. VERSION comes from inputs.version / inputs.version-file and is expanded without double quotes. It should be quoted as "$VERSION".

Locations:

- `action.yml:85`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $FLUXCD_PLUGINS in step "Setup Plugins Dir" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:229`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, static-unsanitized-env-write

**Notes:**

All findings were in action.yml. I quoted $VERSION in the `v*` test. Each value written to $GITHUB_OUTPUT, $GITHUB_PATH or $GITHUB_ENV now goes through `printf '%s' ... | tr -d '\n\r'` first: installed-flux-version, os, arch, the flux tool dir, FLUXCD_PLUGINS and plugin-dir. Each written value comes from a plain variable with no command substitution inside it, so a command that fails still stops the step as it did before. Nothing else changed. I haven't run the action after these edits.

