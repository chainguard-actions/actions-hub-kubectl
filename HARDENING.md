<!-- markdownlint-disable -->

# Hardening Report: actions-hub--kubectl/v1.36.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions-hub--kubectl/v1.36.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks. Failing references: docker-image.yaml: `actions/checkout@v4`, `actions-hub/docker/login@master`, `actions-hub/docker/cli@master`; tests.yaml: `actions/checkout@v4`; upgrader.yaml: `actions/create-release@v1`.

Locations:

- `.github/workflows/docker-image.yaml:12`
- `.github/workflows/docker-image.yaml:15`
- `.github/workflows/docker-image.yaml:24`
- `.github/workflows/tests.yaml:8`
- `.github/workflows/upgrader.yaml:79`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no job within them defines job-level permissions. This means workflows run with the default (potentially broad) GITHUB_TOKEN permissions. All three workflow files are affected: docker-image.yaml, tests.yaml, and upgrader.yaml.

Locations:

- `.github/workflows/docker-image.yaml:1`
- `.github/workflows/tests.yaml:1`
- `.github/workflows/upgrader.yaml:1`

### script-injection (severity: high)

Rule (a): `${{ secrets.GH_TOKEN }}` is directly interpolated inside `run:` shell command strings in upgrader.yaml. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it. Offending lines: (1) `git config --global url."https://${{ secrets.GH_TOKEN }}:@github.com/"...` in the 'Checkout repo' step; (2) the same pattern in the 'Modify Dockerfile' step. The token should be passed via an `env:` block and referenced as `$GH_TOKEN` in the shell.

Locations:

- `.github/workflows/upgrader.yaml:10`
- `.github/workflows/upgrader.yaml:57`

### github-env-injection (severity: high)

Unsanitized values are written to $GITHUB_ENV without applying `tr -d '\n\r'` sanitization, enabling environment variable injection attacks via newline characters.

(1) upgrader.yaml 'Check if new version exist' step: `SDK_VERSION` and `LATEST_VERSION` are fetched from external GitHub API responses via curl/jq/sed and written directly to $GITHUB_ENV (`echo "SDK_VERSION=${SDK_VERSION}" >> $GITHUB_ENV` and `echo "LATEST_VERSION=${LATEST_VERSION}" >> $GITHUB_ENV`) without sanitization. A malicious API response containing newlines could inject arbitrary environment variables.

(2) entrypoint.sh: The variable `$dest` (populated from `inputs.redirect-to` via the `env: dest: ${{ inputs.redirect-to }}` mapping in action.yml) is written to $GITHUB_ENV as a heredoc delimiter (`echo "$dest<<$EOF" >> $GITHUB_ENV`) without sanitization. An attacker-controlled `redirect-to` input containing newlines could break out of the heredoc and inject arbitrary environment variables.

Locations:

- `.github/workflows/upgrader.yaml:36`
- `.github/workflows/upgrader.yaml:37`
- `entrypoint.sh:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 (docker-image.yaml, tests.yaml)
   - actions-hub/docker/login@master → @f5fdbfc3f9d2a9265ead8962c1314108a7b7ec5d (docker-image.yaml)
   - actions-hub/docker/cli@master → @f5fdbfc3f9d2a9265ead8962c1314108a7b7ec5d (docker-image.yaml)
   - actions/create-release@v1 → @0cb9c9b65d5d1901c1f53e5e66eaf4afd303e70e (upgrader.yaml)

2. missing-permissions: Added `permissions: {}` top-level block to all three workflow files.

3. script-injection: In upgrader.yaml, moved `${{ secrets.GH_TOKEN }}` from inline run: shell strings into env: blocks (as GH_TOKEN) for both the 'Checkout repo' and 'Modify Dockerfile' steps.

4. github-env-injection: 
   - upgrader.yaml: Sanitized SDK_VERSION and LATEST_VERSION with `printf '%s' | tr -d '\n\r'` before writing to $GITHUB_ENV.
   - entrypoint.sh: Sanitized $dest with `printf '%s' | tr -d '\n\r'` before using as heredoc delimiter in $GITHUB_ENV.

