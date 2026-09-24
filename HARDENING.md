<!-- markdownlint-disable -->

# Hardening Report: actions-hub--kubectl/v1.37.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions-hub--kubectl/v1.37.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable $dest (set from inputs.redirect-to via `env: dest: ${{ inputs.redirect-to }}` in action.yml) is written directly to $GITHUB_ENV on line 44 without sanitization: `echo "$dest<<$EOF" >> $GITHUB_ENV`. A caller can supply a value containing newline characters to inject arbitrary key=value pairs into the runner's environment, potentially overwriting sensitive environment variables for subsequent steps. The required sanitization step (`printf '%s' "$dest" | tr -d '\n\r'`) is absent before the write.

Locations:

- `entrypoint.sh:44`
- `action.yml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in entrypoint.sh line 44. Added sanitization of the $dest variable (sourced from inputs.redirect-to) before writing it to $GITHUB_ENV. The fix introduces `safe_dest=$(printf '%s' "$dest" | tr -d '\n\r')` to strip newline and carriage return characters, then uses `$safe_dest` in the heredoc delimiter line (`echo "$safe_dest<<$EOF" >> $GITHUB_ENV`) instead of the raw `$dest`. This prevents newline injection attacks that could overwrite sensitive environment variables for subsequent steps. The fix uses POSIX-compatible commands appropriate for the #!/bin/sh shebang.

