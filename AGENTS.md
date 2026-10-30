# AGENTS.md

## Voice

Use Simplified Technical English in formal, operational, and other sensible components of the document where you’re establishing specifics.
Use Plain Language in introductory, expository, friendly, and other sensible components of the document where you’re drawing the reader in or keeping them engaged.

## Repository layout

- `default.json` is the preset that SolZero repositories extend with `github>SolZeroAI/renovate-config`.
- `.github/workflows/renovate.yml` is the reusable workflow that runs Renovate on the calling repository.
- `.github/workflows/self.yml` runs Renovate on this repository. `renovate.json` extends the preset and tracks the pinned Renovate version.
- `README.md` documents each preset setting. Update it when you change `default.json`.

## Rules

- A change here reaches every SolZero repository on its next Renovate run. Keep repository-specific rules in that repository's `renovate.json`.
- Keep the `nub install` command in `default.json` and `RENOVATE_ALLOWED_COMMANDS` in `renovate.yml` identical.
- Do not use `--no-frozen-lockfile` in `postUpgradeTasks`. It re-resolves the whole lock and makes dependency pull requests conflict.
- This repository is public. Do not add secrets, internal hostnames, or process notes.

## Validation

Run before handoff:

```sh
npx --yes --package renovate@44.188.2 -- renovate-config-validator --strict default.json renovate.json
```
