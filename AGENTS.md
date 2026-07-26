# Repository instructions

## Communication

- Communicate with the repository owner in Traditional Chinese using Taiwan
  terminology.
- Keep repository files and external-facing text in English.
- Preserve Renovate's official option names exactly.

## Repository role

- Forgejo `git.xlion.tw/xlionjuan/renovatebot-presets` is the only source
  repository.
- GitHub `xlionjuan/renovatebot-presets` is an automatic mirror. Never edit or
  push to it directly.
- Consumers on both platforms use
  `local>xlionjuan/renovatebot-presets`.
- Consumers do not pin preset tags, so changes on `main` have broad impact.

## Files and ownership

- `default.json` contains only policy shared across repositories.
- `go.json` is an additive Go policy and must not extend the default preset
  itself.
- `renovate.json` manages only this source repository. The GitHub
  mirror must not be processed by Renovate.
- `.forgejo/workflows/validate.yml` is the preset compatibility gate.
- `README.md` is the owner's maintenance guide. Update it whenever behavior or
  policy boundaries change.

## Branch policy

- Maintain this preset repository directly on `main`, as explicitly required by
  the owner.
- Push only to the Forgejo source. Never push the GitHub mirror.
- Consumer repository Renovate config changes are different: make them on
  `renovate/reconfigure` and push that branch directly to the source
  repository, not from a fork.
- Open a PR for GitHub consumers. For Forgejo consumers, push the branch without
  opening a PR.

## Configuration style

- Use the standard filenames `default.json` and `go.json`, and keep them valid
  strict JSON.
- Use `description` to explain non-obvious behavior. Do not rely on rule order
  that a maintainer must infer.
- Do not add `enabledManagers`. Let Renovate detect managers and handle
  exceptions with precise rules.
- Do not add options that merely restate current Renovate defaults unless the
  owner explicitly decides to pin a safety invariant.
- Do not add a top-level schedule for normal dependencies. The weekly lock-file
  maintenance inherited from `config:best-practices` is an accepted exception.
- Do not add global `ignoreTests: true`.
- Scope automerge by package namespace or name, manager, and update type.
- Major updates must not automerge by default.
- Do not promote repository-specific package, path, workflow, CI, or schedule
  exceptions into a preset unless the owner confirms they are a
  cross-repository contract.
- General custom managers must require an explicit opt-in marker.

## Compatibility

- The deployed self-hosted Renovate version is the compatibility baseline.
- Current documentation may describe features that the deployed bot does not
  support. Before adopting a feature, inspect source or versioned documentation
  for the matching Renovate tag.
- Brief version drift between the validator and production bot is accepted. A
  green CI result proves compatibility only with the validator version pinned
  in the workflow.
- Renovate major updates must not automerge. Before upgrading, re-check
  experimental features, beta managers, deprecations, config migrations, and
  native-manager changes.

## Validation

Every configuration change must pass:

```sh
renovate-config-validator \
  --strict \
  --no-global \
  renovate.json \
  default.json \
  go.json
```

- Never omit `--no-global`. The official validator otherwise treats explicitly
  named files as self-hosted global configuration.
- Run `git diff --check`.
- Confirm that JSON parses, workflow YAML parses, and no unintended
  repository-specific policy entered the default preset.
- After pushing preset `main`, wait for the pinned-version Forgejo validator.
  Fix or revert immediately if it fails.

## Maintenance and rollout

- Change and validate the Forgejo source first, then confirm that the GitHub
  mirror has synchronized.
- Before broad consumer rollout, validate `local>` resolution with at least one
  GitHub repository and one Forgejo repository.
- If a change causes widespread problems, revert the preset commit instead of
  adding conflicting temporary overrides to individual consumers.
- Do not modify consumer repository configuration from this repository.
