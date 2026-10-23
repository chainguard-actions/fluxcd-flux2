<!-- markdownlint-disable -->

# Hardening Report: fluxcd--flux2--action/v2.8.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fluxcd--flux2--action/v2.8.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Download the binary to the runner's cache dir' writes FLUX_TOOL_DIR to $GITHUB_PATH without the required sanitization (`echo "$FLUX_TOOL_DIR" >> "$GITHUB_PATH"`). FLUX_TOOL_DIR comes from `inputs.bindir` through env:, and the default path also contains VERSION, which comes from `inputs.version`. A newline in either value could add extra PATH entries. Fix: safe=$(printf '%s' "$FLUX_TOOL_DIR" | tr -d '\n\r') before the write.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Sub-rule (b): an input-derived env var is expanded without quotes. Offending line: `if [[ $VERSION = v* ]]; then`. VERSION comes from `inputs.version`. Bash's [[ ]] does not split words on its left side, so the practical risk is low, but the rule says every such expansion must be quoted, like the `case $VAR` example it gives. Fix: use `[[ "$VERSION" = v* ]]`.

Locations:

- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Quoted $VERSION in the [[ ]] v-prefix test. Before writing FLUX_TOOL_DIR to $GITHUB_PATH, it is now stripped of newlines and carriage returns with printf | tr -d '\n\r'.

