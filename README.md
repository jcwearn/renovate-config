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

`config:best-practices` already enables weekly lock file maintenance via `:maintainLockFilesWeekly`,
so repos don't need to declare `lockFileMaintenance` themselves.

Do **not** add a top-level `minimumReleaseAge` here: `pin`, `bump`, `lockfileUpdate`, `rollback`,
`replacement` and `lockFileMaintenance` updates carry no release timestamp, so with the default
`internalChecksFilter: "strict"` they'd be withheld permanently. Use the scoped
`security:minimumReleaseAge*` presets, which carry the necessary opt-out rules.
