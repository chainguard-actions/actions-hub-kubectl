<!-- markdownlint-disable -->

# Hardening Report: actions-hub--kubectl/v1.35.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions-hub--kubectl/v1.35.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the shell variable `$dest` is populated from `inputs.redirect-to` via the action.yml `env:` block (`dest: ${{ inputs.redirect-to }}`), making it attacker-controlled. It is written directly to `$GITHUB_ENV` without sanitization: `echo "$dest<<$EOF" >> $GITHUB_ENV`. An attacker can inject newlines into the `redirect-to` input to poison the GitHub environment file (e.g., inject arbitrary key=value pairs into the runner environment). The required sanitization step (`printf '%s' "$dest" | tr -d '\n\r'`) is absent before the write.

Locations:

- `entrypoint.sh:35`
- `action.yml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in entrypoint.sh by sanitizing the $dest variable before writing it to $GITHUB_ENV. Added `safe_dest=$(printf '%s' "$dest" | tr -d '\n\r')` and replaced `echo "$dest<<$EOF" >> $GITHUB_ENV` with `echo "$safe_dest<<$EOF" >> $GITHUB_ENV`. This strips newlines and carriage returns from the attacker-controlled `redirect-to` input before it is used as a heredoc key name in the GitHub environment file, preventing environment file poisoning attacks.

