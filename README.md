# renovate-config

This repository keeps one Renovate setup for every SolZero repository. A repository extends the preset and calls the shared workflow, so a change here reaches the whole fleet.

## Use it in a repository

1. Add `renovate.json`:

   ```json
   {
     "$schema": "https://docs.renovatebot.com/renovate-schema.json",
     "extends": ["github>SolZeroAI/renovate-config"]
   }
   ```

2. Add `.github/workflows/renovate.yml`:

   ```yaml
   name: Renovate

   on:
     schedule:
       - cron: "17 6 * * *"
     push:
       branches: [master]
       paths: ["**/package.json", "nub.lock"]
     workflow_dispatch:

   permissions:
     contents: read

   concurrency:
     group: renovate
     cancel-in-progress: false

   jobs:
     renovate:
       uses: SolZeroAI/renovate-config/.github/workflows/renovate.yml@master
       secrets: inherit
   ```

3. Make sure the repository has:
   - The `solzero-bot` GitHub App installed, and the org secrets `RELEASE_APP_CLIENT_ID` and `RELEASE_APP_PRIVATE_KEY`.
   - **Allow auto-merge** turned on in the repository settings.
   - Required status checks in the default-branch ruleset. Auto-merge waits for these checks. With no required checks, a pull request merges at once.
   - Dependabot alerts on and Dependabot security updates off. Renovate reads the alerts and opens the security pull requests itself.
   - A `.nvmrc` file. The workflow reads the Node version from it.

Pass `with: { runs-on: ubuntu-latest }` to the workflow call in a repository that has no Blacksmith runners.

## What the preset does

| Setting | Effect |
| --- | --- |
| `automerge`, `platformAutomerge` | Renovate turns on GitHub auto-merge. The pull request merges when the required checks pass. |
| Major updates | Auto-merge is off. A human merges them. |
| Minor, patch, pin and digest updates | Grouped into one `non-major dependencies` pull request each Monday before 06:00 UTC. The oxc toolchain gets its own group. |
| Security updates | Open at once, separately, and auto-merge when green. |
| `minimumReleaseAge` | An npm release must be 3 days old before Renovate proposes it. Security fixes skip this wait. |
| `postUpgradeTasks` | Runs `nub install --prefer-frozen-lockfile --lockfile-only --ignore-scripts`, which changes only the `nub.lock` entries for the updated packages. |
| `rebaseWhen: conflicted` | Renovate rebuilds a branch only when it conflicts. The rebuild regenerates `nub.lock`, so no human resolves a lock conflict. |
| GitHub Actions | Stay on major tags such as `actions/checkout@v7`. Renovate does not pin them to exact versions or SHAs. |
| Labels | `dependencies` on all, `security` on security updates, `release:none` on dev dependencies and GitHub Actions. |

The caller workflow also runs Renovate after each push that changes a manifest or `nub.lock`. When one dependency pull request merges, Renovate rebuilds the others that now conflict, and auto-merge takes them in turn.

### Why `--prefer-frozen-lockfile`

`nub install --no-frozen-lockfile` re-resolves every range in the workspace. In platform it rewrote about 600 lines of `nub.lock` with no manifest change. Each Renovate pull request then rewrote the same lines, so the pull requests conflicted with each other, and the churn split peer variants (for example two `McpServer` types). `--prefer-frozen-lockfile` keeps the existing resolutions and resolves only what the manifest change needs.

## Validation

`Validate` runs `renovate-config-validator --strict` on each pull request and push.
