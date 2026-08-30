# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) config preset used by all repos managed by the
self-hosted Renovate bot.

## Usage

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>jcwearn/renovate-config"]
}
```

The bot writes exactly this file into its onboarding PR (via `onboardingConfig`), so new repos pick
it up with no hand-editing.

## What it sets

`config:best-practices` plus:

- `security:minimumReleaseAgePypi` — 3-day delay before raising PyPI updates, matching what
  best-practices already does for npm. Matters because non-major updates are automerged.
- `security:openssf-scorecard` — OpenSSF Scorecard badge column in PR bodies.
- `postUpdateOptions` — `npmDedupe`, `pnpmDedupe`, `gomodTidy`, `gomodUpdateImportPaths`. Each is
  read only by its own package manager, so all are inert on repos that don't use them.

- `customManagers` — one regex manager for the `# renovate:` annotation convention: a comment of the
  form `# renovate: datasource=<ds> depName=<name>` makes Renovate update the quoted value on the
  next line. `versioning=` and `extractVersion=` are picked up from the same comment when present.

  Renovate reads nothing from these comments on its own, so before this they were inert. Three
  `tofu_version: "1.12.6"` pins in `truenas-infra`'s workflows carried the annotation and had never
  produced a single PR.

  The capture deliberately steps over a leading `= `, so a Terraform pin written
  `version = "= 3.0.0"` keeps its operator across an update. The built-in `terraform` manager cannot:
  it round-trips the constraint through npm versioning, which has no `=` operator, so it rewrites
  `= 2.4.1` as a bare `3.0.0`. A repo that pins that way should disable the built-in manager for the
  file, or the two managers will fight over it.

`config:best-practices` already enables weekly lock file maintenance via `:maintainLockFilesWeekly`,
so repos don't need to declare `lockFileMaintenance` themselves.

Do **not** add a top-level `minimumReleaseAge` here: `pin`, `bump`, `lockfileUpdate`, `rollback`,
`replacement` and `lockFileMaintenance` updates carry no release timestamp, so with the default
`internalChecksFilter: "strict"` they'd be withheld permanently. Use the scoped
`security:minimumReleaseAge*` presets, which carry the necessary opt-out rules.
