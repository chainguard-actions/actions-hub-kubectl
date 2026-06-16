<!-- markdownlint-disable -->

# Hardening Report: actions-hub--kubectl/v1.36.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions-hub--kubectl/v1.36.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable `$dest` (populated from `inputs.redirect-to` via `env: dest: ${{ inputs.redirect-to }}` in action.yml) is written directly to `$GITHUB_ENV` on line 43 without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A caller-controlled value containing newlines can inject arbitrary environment variable assignments into the runner's environment. The offending line is: `echo "$dest<<$EOF" >> $GITHUB_ENV`

Locations:

- `entrypoint.sh:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in entrypoint.sh line 43. The caller-controlled variable `$dest` (from `inputs.redirect-to`) was written directly to `$GITHUB_ENV` without sanitization. Added a sanitization step: `safe_dest=$(printf '%s' "$dest" | tr -d '\n\r')` to strip newlines and carriage returns before using the value in the heredoc delimiter line and the `add-mask` command. This prevents newline injection attacks that could inject arbitrary environment variable assignments into the runner's environment.

