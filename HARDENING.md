# Hardening Report: fluxcd--flux2--action/v2.8.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `c40cfe5fa14e08549b1b988e7e5a26da4816abf0`

**Test Policy SHA:** `f2e7d85641cde4267138117189b8eba7ba2bfbde`

Action **fluxcd--flux2--action/v2.8.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In action.yml, the attacker-controlled input `inputs.bindir` is assigned to the environment variable `FLUX_TOOL_DIR` (env: block, line 27) and then written directly to `$GITHUB_PATH` via `echo "$FLUX_TOOL_DIR" >> "$GITHUB_PATH"` (line ~130) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A caller-supplied `bindir` value containing embedded newlines could inject arbitrary additional entries into GITHUB_PATH, potentially hijacking the PATH used by subsequent workflow steps. The fix is to sanitize the value before writing: `safe=$(printf '%s' "$FLUX_TOOL_DIR" | tr -d '\n\r') && echo "$safe" >> "$GITHUB_PATH"`

Locations:

- `action.yml:27`
- `action.yml:130`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in hardened/fluxcd--flux2--action/v2.8.3/action.yml. The `FLUX_TOOL_DIR` value (derived from the attacker-controlled `inputs.bindir`) was being written directly to $GITHUB_PATH without sanitization. Added a sanitization step: `safe=$(printf '%s' "$FLUX_TOOL_DIR" | tr -d '\n\r')` and then `echo "$safe" >> "$GITHUB_PATH"` to strip any embedded newline characters before writing to $GITHUB_PATH, preventing PATH injection attacks.

