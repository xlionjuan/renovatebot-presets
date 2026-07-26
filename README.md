# renovatebot-presets

Shared Renovate configuration maintained by `xlionjuan`.
`git.xlion.tw/xlionjuan/renovatebot-presets` is the source repository on
Forgejo and is automatically mirrored to GitHub as
`xlionjuan/renovatebot-presets`.

Repositories on both platforms use `local>` references. Presets are not pinned
to tags; consumers follow the default branch of the local source or mirror.

## Presets

### Default

```json
{
  "extends": ["local>xlionjuan/renovatebot-presets"]
}
```

`default.json` defines policy shared across repositories:

- Extend Renovate's `config:best-practices`.
- Use the `Asia/Taipei` timezone without scheduling normal dependency updates.
- Keep the weekly lock-file maintenance inherited from
  `config:best-practices`.
- Set `platformAutomerge: false`; Renovate itself must observe successful CI
  before merging.
- Enable semantic commits and limit concurrent Renovate PRs to five.
- Apply a three-day cooldown to normal updates without blocking releases that
  have no timestamp forever.
- Pin internal checks to strict behavior.
- Enable experimental OSV vulnerability alerts.
- Add the `dependencies` label with `addLabels`, allowing repository-specific
  labels to be combined with it.
- Enable the beta `git-submodules` manager.
- Support marker-based GitHub Actions `_VERSION` variables and Git ref SHA
  variables.

### Go

```json
{
  "extends": [
    "local>xlionjuan/renovatebot-presets",
    "local>xlionjuan/renovatebot-presets:go"
  ]
}
```

`go.json` is an additive Go policy and currently enables only `gomodTidy`. It
does not extend the default preset itself, so Go repositories must list both
presets.

## Automerge policy

The default preset enables automerge only for non-major updates from these
trusted Action namespaces:

- `actions/*`
- `github/*`
- `xlionjuan/*`

Covered update types are `pin`, `digest`, `pinDigest`, `minor`, and `patch`.
Major updates only open a PR.

Even when a workflow uses an explicit Forgejo URL such as
`https://git.xlion.tw/actions/checkout@...`, Renovate normalizes its
`packageName` to `actions/checkout`. The `actions/*` rule therefore covers both
GitHub's `actions/checkout@...` syntax and the full Forgejo URL form.

The default preset does not set `ignoreTests: true`. Automerge does not occur
without a successful CI check visible to Renovate. Packages that must merge
without CI require a narrowly scoped repository-specific exception.

`ghcr.io/xlionjuan/fedora-createrepo-image` is a global exception. Consumer
workflows must track `latest`, resolve its digest at runtime, validate that
digest, and use the same resolved digest for the rest of the run. Renovate
ignores this dependency completely and must not pin or update it.

## Opt-in version markers

### Action environment versions

The default preset extends Renovate's official
`customManagers:githubActionsVersions` preset. Workflows or action YAML under
`.github`, `.gitea`, or `.forgejo` can use:

```yaml
# renovate: datasource=github-releases depName=anomalyco/opencode versioning=loose
OPENCODE_VERSION: v1.2.3
```

### Git ref pinned to a SHA

Use a branch or tag as the moving target while executing an immutable SHA:

```yaml
# renovate: datasource=git-refs depName=samber/cc-skills-golang packageName=https://github.com/samber/cc-skills-golang currentValue=main
CC_SKILLS_GOLANG_SHA: 0123456789abcdef0123456789abcdef01234567
```

The marker must be immediately followed by a variable ending in `_SHA`, whose
value is exactly 40 lowercase hexadecimal characters. Git SHA and digest
updates generally cannot apply `minimumReleaseAge` reliably, so the three-day
cooldown is best effort for these updates.

## Security and beta features

`osvVulnerabilityAlerts` is experimental. It checks only direct dependencies
in supported ecosystems, excludes GitHub Actions, and can make PRs reappear
when advisory data changes. GitHub-native vulnerability alerts still depend on
platform capabilities and permissions.

The `git-submodules` manager is beta and disabled by default in the deployed
Renovate version. This preset intentionally enables it globally, so every
consumer containing `.gitmodules` is scanned. Automerge or Dependency
Dashboard approval for submodules remains repository-specific.

## What belongs where

| Location | Test | Examples |
| --- | --- | --- |
| `default.json` | It would be surprising for any new personal repository not to inherit the rule. | Timezone, CI-gated automerge policy, shared labels |
| `go.json` | Every root Go module needs it, and it is meaningless outside Go projects. | `gomodTidy` |
| Repository config | The rule concerns a particular package, file, workflow, CI capability, schedule, or risk exception. | No-CI automerge, a special Dockerfile schedule |

Repeated configuration is not automatically shared policy. First verify that
the intent is the same in every repository. A general custom manager may live
in the default preset when it requires an explicit opt-in marker.

Do not add:

- `enabledManagers`; let Renovate detect managers and use precise package rules
  for exceptions.
- Options that merely restate Renovate's current defaults.
- Schedules without a specific repository-level reason.
- Global `ignoreTests: true` or unscoped automerge.

## Validation

CI runs a Renovate container pinned by tag and digest, and follows Renovate's
requirement to validate preset files as non-global configuration:

```sh
renovate-config-validator \
  --strict \
  --no-global \
  .forgejo/renovate.json \
  default.json \
  go.json
```

The validator and production bot pin their versions independently, and brief
version drift is accepted. Both Renovate containers update on Wednesdays
without a cooldown. Non-major updates automerge only after their complete CI
passes; major updates require manual review. Before using a new option, confirm
that the deployed bot version supports it instead of relying only on the latest
documentation.

## Maintenance workflow

This preset repository is maintained directly on `main`:

1. Pull the latest `main`.
2. Edit the presets. Add a `description` and update this README for risky or
   non-obvious behavior.
3. Run the local static checks and available validator.
4. Commit and push `main` to the Forgejo source.
5. Wait for the pinned-version validator workflow. Fix or revert immediately
   if it fails.
6. Confirm that the GitHub mirror is synchronized before GitHub consumers adopt
   the change.
7. Observe the first Renovate run and Dependency Dashboard on both platforms.

Consumer repository configuration changes follow Renovate's special validation
workflow instead: push them directly to a `renovate/reconfigure` branch in the
source repository. Open a PR for GitHub consumers; for Forgejo consumers, push
the branch without opening a PR.

If a change causes widespread problems, revert the preset commit. Consumers
that follow the untagged default branch will resolve the reverted policy on
their next Renovate run.

Before a Renovate major upgrade, re-check:

- Experimental features and beta managers.
- Changes to Renovate defaults.
- Custom managers that native managers can now replace.
- Deprecated options or required config migrations.
- Validator results from the version that will be deployed.
