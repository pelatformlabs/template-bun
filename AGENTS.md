# AGENTS.md

## Quick Rules

- **Always use `bun`, never `npm`/`yarn`.** `packageManager` is `bun@1.3.14`.
- Engines: `node >= 22`, `bun >= 1.3.0`.
- Workspaces: `packages/**`, `apps/**`, `examples/**` (defined in both `package.json` and `bunfig.toml`).
- Internal deps use workspace protocol: `"@pelatform/other": "workspace:*"`.
- `bun install` runs `prepare` → installs Husky hooks (v9, `.husky/_/`).

## Commands

```bash
bun install                 # also installs husky hooks
bun run build               # turbo build (dependsOn: ^build)
bun run types:check         # turbo types:check (dependsOn: ^build)
bun run lint                # biome check + turbo run lint
bun run lint:fix            # biome check --write --unsafe + turbo run lint -- --fix
bun run format              # biome format --write
bun run dev                 # turbo dev (persistent)
bun run clean               # turbo clean
bun run clean:all           # turbo clean + rm .husky .turbo bun.lock node_modules
bun run test                # turbo test
npx changeset               # create a changeset
bun run version             # changeset version + bun update
bun run release             # bash scripts/publish.sh (publishes only packages/**, not apps/ or examples/)
```

Filter to a single workspace:

```bash
bun run build --filter=@pelatform/core
bun run test --filter=@pelatform/main
bun add <pkg> --filter=@pelatform/core
```

## Pipeline (turbo.json)

| Task            | dependsOn | Persistent | Cached |
| --------------- | --------- | ---------- | ------ |
| build           | ^build    | no         | yes    |
| types:check     | ^build    | no         | yes    |
| dev             | -         | yes        | no     |
| start           | ^build    | yes        | no     |
| lint            | -         | no         | yes    |
| clean/clean:all | -         | no         | no     |

## Lint-staged (pre-commit)

- `*.{js,ts,cjs,mjs,...}` → `biome check --write --no-errors-on-unmatched`
- `*.{md,yml,yaml}` → `prettier --write`
- `*.{json,jsonc,html}` → `biome format --write --no-errors-on-unmatched`

## Commit Convention

Commitlint enforces types: `feat`, `feature`, `fix`, `refactor`, `docs`, `build`, `test`, `ci`, `chore`. Configured in `.commitlintrc.cjs`. Format: `type(scope?): message`.

## Changesets

- Repo: `pelatformlabs/template-bun`
- `commit: false` — changeset PRs are auto-committed by CI, not locally.
- `access: public`, `baseBranch: main`
- `bumpVersionsWithWorkspaceProtocolOnly: true`

## CI

- **Lint** (`.github/workflows/lint.yml`): PRs to `main` → build → lint:fix → lint → types:check
- **Release** (`.github/workflows/release.yml`): push to `main` with `.changeset/**` or `packages/**` changes → build → lint → changesets/action (version PR or publish). Uses `GH_PAT || GITHUB_TOKEN` and `NPM_TOKEN`.

## Biomes

Extends `@pelatform/biome-config/base` (`biome.jsonc`).
