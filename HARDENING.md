<!-- markdownlint-disable -->

# Hardening Report: actions-hub--kubectl/v1.36.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions-hub--kubectl/v1.36.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable `$dest` — which is populated from `inputs.redirect-to` via the action.yml env mapping (`dest: ${{ inputs.redirect-to }}`) — is written directly to `$GITHUB_ENV` without sanitization. Specifically, line 43 executes `echo "$dest<<$EOF" >> $GITHUB_ENV`. Because `$dest` is an inherited process env var set by the calling workflow from user-controlled input, an attacker can supply a value containing newline characters to inject arbitrary key=value pairs into the runner's environment. The required sanitization step (`safe=$(printf '%s' "$dest" | tr -d '\n\r')`) is absent before the write.

Locations:

- `entrypoint.sh:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in entrypoint.sh line 43. Added sanitization of the $dest variable (which comes from user-controlled input `inputs.redirect-to`) before writing it to $GITHUB_ENV. The fix introduces `safe_dest=$(printf '%s' "$dest" | tr -d '\n\r')` to strip newline and carriage return characters, then uses `$safe_dest` instead of `$dest` in the heredoc delimiter line (`echo "$safe_dest<<$EOF" >> $GITHUB_ENV`). This prevents an attacker from injecting arbitrary environment variable key=value pairs by supplying newline characters in the redirect-to input.

