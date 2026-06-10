<!-- markdownlint-disable -->

# Hardening Report: actions-hub--kubectl/v1.35.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions-hub--kubectl/v1.35.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable `$dest` is sourced from the `inputs.redirect-to` action input (set via `env: dest: ${{ inputs.redirect-to }}` in action.yml) and written directly to `$GITHUB_ENV` on line 42 without sanitization: `echo "$dest<<$EOF" >> $GITHUB_ENV`. An attacker-controlled value containing newline characters could inject arbitrary key=value pairs into the GitHub environment file, allowing environment variable hijacking. The required sanitization step (`safe=$(printf '%s' "$dest" | tr -d '\n\r')`) is absent before this write.

Locations:

- `entrypoint.sh:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in entrypoint.sh line 42. Added `safe_dest=$(printf '%s' "$dest" | tr -d '\n\r')` to strip newline and carriage return characters from the user-controlled `$dest` variable before writing it to $GITHUB_ENV as a heredoc delimiter. Replaced all uses of `$dest` in the else branch with `${safe_dest}` to ensure the sanitized value is used consistently.

