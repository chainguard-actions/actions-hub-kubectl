<!-- markdownlint-disable -->

# Hardening Report: actions-hub--kubectl/v1.36.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions-hub--kubectl/v1.36.4** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

entrypoint.sh writes the `$dest` variable (sourced from `inputs.redirect-to` via action.yml's `env: dest: ${{ inputs.redirect-to }}`) to $GITHUB_ENV without sanitization. The line `echo "$dest<<$EOF" >> $GITHUB_ENV` allows an attacker-controlled value containing newlines to inject arbitrary environment variable assignments into the runner environment. The required sanitization step (`printf '%s' "$dest" | tr -d '\n\r'`) is absent.

Locations:

- `entrypoint.sh:37`
- `action.yml:14`

### script-injection (severity: high)

upgrader.yaml interpolates `${{ secrets.GH_TOKEN }}` directly inside two `run:` shell command strings (sub-rule a). Any `${{ ... }}` expression embedded in a `run:` block is subject to YAML template substitution before the shell processes it, creating a script-injection risk. Offending lines: `git config --global url."https://${{ secrets.GH_TOKEN }}:@github.com/".insteadOf "https://github.com/"` (appears in the 'Checkout repo' step and the 'Modify Dockerfile' step). The value should be passed via an `env:` variable and referenced as `$GH_TOKEN` in the shell.

Locations:

- `.github/workflows/upgrader.yaml:12`
- `.github/workflows/upgrader.yaml:60`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tag or branch names instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised.

docker-image.yaml:
  - uses: actions/checkout@v4
  - uses: actions-hub/docker/login@master
  - uses: actions-hub/docker/cli@master

tests.yaml:
  - uses: actions/checkout@v4

upgrader.yaml:
  - uses: actions/create-release@v1

Locations:

- `.github/workflows/docker-image.yaml:13`
- `.github/workflows/docker-image.yaml:17`
- `.github/workflows/docker-image.yaml:28`
- `.github/workflows/tests.yaml:9`
- `.github/workflows/upgrader.yaml:68`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual job within them defines a `permissions:` key either. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g., write access to contents, packages, etc.). All three files should declare minimal required permissions.

Locations:

- `.github/workflows/docker-image.yaml:1`
- `.github/workflows/tests.yaml:1`
- `.github/workflows/upgrader.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings: (1) github-env-injection in entrypoint.sh by sanitizing $dest with `printf '%s' "$dest" | tr -d '\n\r'` before writing to GITHUB_ENV; (2) script-injection in upgrader.yaml by moving both occurrences of `${{ secrets.GH_TOKEN }}` from run: shell strings into env: blocks and referencing as ${GH_TOKEN}; (3) unpinned-uses by pinning all four action references to full 40-char SHAs (actions/checkout@11d5960..., actions-hub/docker/login@f5fdbfc..., actions-hub/docker/cli@f5fdbfc..., actions/create-release@0cb9c9b...); (4) missing-permissions by adding top-level permissions blocks to all three workflow files with minimal required permissions (docker-image.yaml: contents:read+packages:write, tests.yaml: contents:read, upgrader.yaml: contents:write).

