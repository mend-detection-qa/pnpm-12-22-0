# lockfile-v13-file-catalog

Probe targeting `lockfile-v13-file-catalog` from the pnpm
feature-coverage catalog. Added from pnpm 12.22.0 release
(2026-11-17).

## Feature exercised

Three interlocking pnpm 12.22.0 features exercised together:
(1) experimental v13 lockfile format with per-importer
`specifiers:`/`dependencies:` split and top-level `catalogs:`
block; (2) `file:` local dependency whose own `package.json`
uses `catalog:` specifiers resolved through the workspace
catalog; (3) registry re-verification of lockfile resolutions
when the registry URL changes.

## Pattern

`lockfile-v13-file-catalog` from
`plugins/js-pnpm/skills/pnpm-core/references/feature-coverage-patterns.md`

## Project structure

```
.
├── package.json              root — depends on zod (catalog:)
│                             and @probe/local-lib (file:)
├── pnpm-lock.yaml            v13 format (lockfileVersion: '13.0')
├── pnpm-workspace.yaml       defines catalog: { zod, ms }
├── .npmrc                    experimental-lockfile-version=13
├── .whitesource              Bucket A versioning pins
├── expected-tree.json
└── packages/
    └── local-lib/
        └── package.json      depends on ms (catalog:)
```

## Expected dependency tree

- Root direct deps:
  - `zod@3.22.4` — registry, resolved from default catalog
  - `@probe/local-lib@1.0.0` — local, path `./packages/local-lib`
- `@probe/local-lib` direct deps:
  - `ms@2.1.3` — registry, resolved from default catalog
    through the `file:` boundary (key new behavior in 12.22.0)
- `zod` and `ms` have no transitive deps at these versions.
- All packages: `group: "main"`.

### Key Mend detection challenges

1. Parser must handle the v13 `specifiers:`/`dependencies:`
   split — values in `dependencies:` are bare strings, not
   `{specifier, version}` objects.
2. `catalog:` literals must be resolved via `catalogs.default`
   in the lockfile, not retained as the version string.
3. `@probe/local-lib` must be reported as `source: "local"`,
   not as a registry package.
4. `ms` (a dep of the local lib resolved via `catalog:`) must
   appear in the tree as a child of `@probe/local-lib`.

## Mend config

Bucket A — default-emit with `pnpm` pinned to `12.22.0` and
`node` pinned to `20.11.1`. The experimental v13 lockfile
format is enabled via `.npmrc: experimental-lockfile-version=13`.
No `whitesource.config` needed; `configMode` is `AUTO`.

## Probe metadata

| Key                   | Value                          |
|-----------------------|--------------------------------|
| pattern               | lockfile-v13-file-catalog      |
| pm                    | pnpm                           |
| pm_version_under_test | 12.22.0                        |
| schema_version        | 1.2                            |
| target                | local                          |
| generated_at          | 2026-11-17T20:36:37Z           |
| categories            | lockfile_format, tree_structure, version_constraints |
