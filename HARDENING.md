<!-- markdownlint-disable -->

# Hardening Report: actions-hub--kubectl/v1.37.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions-hub--kubectl/v1.37.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable `$dest` (sourced from `inputs.redirect-to` via `env: dest: ${{ inputs.redirect-to }}` in action.yml) is written directly to $GITHUB_ENV without sanitization: `echo "$dest<<$EOF" >> $GITHUB_ENV`. An attacker-controlled value for `inputs.redirect-to` containing newlines could inject arbitrary environment variables. The required sanitization step (`printf '%s' "$dest" | tr -d '\n\r'`) is absent before the write.

Locations:

- `entrypoint.sh:38`
- `action.yml:14`

### script-injection (severity: high)

In upgrader.yaml, `${{ secrets.GH_TOKEN }}` is interpolated directly inside two `run:` shell command strings (sub-rule a). Any ${{ ... }} expression inside a run: block is a script-injection risk because the value is substituted into the shell command string before the shell parses it. Affected lines: `git config --global url."https://${{ secrets.GH_TOKEN }}:@github.com/".insteadOf "https://github.com/"` (appears in two separate steps).

Locations:

- `.github/workflows/upgrader.yaml:11`
- `.github/workflows/upgrader.yaml:68`

### unpinned-uses (severity: high)

Multiple workflow files use `uses:` references pinned to mutable tags or branch names instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks. Unpinned references found:
- docker-image.yaml: `actions/checkout@v4`, `actions-hub/docker/login@master`, `actions-hub/docker/cli@master`
- tests.yaml: `actions/checkout@v4`
- upgrader.yaml: `actions/create-release@v1`

Locations:

- `.github/workflows/docker-image.yaml:12`
- `.github/workflows/docker-image.yaml:16`
- `.github/workflows/docker-image.yaml:26`
- `.github/workflows/tests.yaml:7`
- `.github/workflows/upgrader.yaml:76`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no job within them defines job-level permissions either. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All three files — docker-image.yaml, tests.yaml, and upgrader.yaml — are affected.

Locations:

- `.github/workflows/docker-image.yaml:1`
- `.github/workflows/tests.yaml:1`
- `.github/workflows/upgrader.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings:
1. github-env-injection (entrypoint.sh:38): Added `safe_dest=$(printf '%s' "$dest" | tr -d '\n\r')` and used `$safe_dest` instead of `$dest` when writing the GITHUB_ENV heredoc key, preventing newline injection.
2. script-injection (upgrader.yaml:11,68): Moved `${{ secrets.GH_TOKEN }}` out of both `run:` shell strings into `env: GH_TOKEN: ${{ secrets.GH_TOKEN }}` blocks, then referenced `${GH_TOKEN}` in the shell scripts.
3. unpinned-uses: Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262, actions-hub/docker/login@master → @f5fdbfc3f9d2a9265ead8962c1314108a7b7ec5d, actions-hub/docker/cli@master → @f5fdbfc3f9d2a9265ead8962c1314108a7b7ec5d, actions/create-release@v1 → @0cb9c9b65d5d1901c1f53e5e66eaf4afd303e70e.
4. missing-permissions: Added `permissions: contents: read, packages: write` to docker-image.yaml; `permissions: contents: read` to tests.yaml; `permissions: contents: write` to upgrader.yaml (needs write for git push and release creation).

