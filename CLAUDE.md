# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Bun monorepo template for building and publishing TypeScript/Node packages. It uses:

- **Bun** as the package manager (required: `bun >= 1.0.0`)
- **Node.js** (required: `node >= 22`)
- **Turborepo** for task orchestration across workspaces
- **Biome** for linting and formatting (extends `@pelatform/biome-config/base`)
- **Changesets** for versioning and publishing
- **Husky** for Git hooks
- **Commitlint** with `@commitlint/config-conventional` for commit message linting

**Important**: This project uses Bun as the package manager (`packageManager: bun@1.3.5`). Always use `bun` commands instead of `npm` or `yarn`.

## Common Commands

```bash
# Install dependencies
bun install

# Development (persistent tasks across all workspaces)
bun run dev

# Build all workspaces
bun run build

# Type-check all workspaces
bun run types:check

# Lint (check only - runs biome check + turbo run lint)
bun run lint

# Lint with auto-fix (runs biome check --write --unsafe + turbo run lint -- --fix)
bun run lint:fix

# Format code with Biome
bun run format

# Clean build artifacts
bun run clean

# Deep clean (removes node_modules, .husky, .turbo, bun.lock)
bun run clean:all

# Create a changeset for versioning
npx changeset

# Version packages (updates versions and runs bun update)
bun run version

# Publish packages (via scripts/publish.sh)
bun run release
```

## Running Tasks in Specific Workspaces

Use Turbo's `--filter` flag to run tasks in specific packages:

```bash
# Build a specific package
bun run build --filter=@pelatform/core

# Run dev in a specific workspace
bun run dev --filter=@pelatform/web

# Run tests for a specific package
bun run test --filter=@pelatform/main
```

## Workspace Structure

```
packages/           # Published or internal packages
├── core/          # Core package
└── main/          # Main package

apps/              # Optional applications consuming packages
├── web/           # Web application
└── docs/          # Documentation
```

Workspaces are defined in both `package.json` and `bunfig.toml`. Workspaces follow the pattern `packages/**` and `apps/**`, allowing for a flexible monorepo structure where `packages/` contains publishable libraries and `apps/` contains applications that consume those packages.

## Turborepo Pipeline

The `turbo.json` defines task dependencies:

- `build`: Depends on `^build` (build dependencies first). Outputs: `.next/**`, `.source/**`, `build/**`, `dist/**`, `out/**`
- `dev`: Persistent, uncached
- `start`: Depends on `^build`, persistent
- `lint`: No dependencies
- `types:check`: Depends on `^build`
- `clean`/`clean:all`: Uncached, non-persistent

## Code Quality

**Biome** (`biome.jsonc`) extends `@pelatform/biome-config/base` for consistent linting and formatting across the project. Lint-staged with Husky pre-commit hooks ensures code quality before commits.

**Lint-staged configuration**:

- TypeScript/JavaScript: Biome check with auto-fix
- Markdown/YAML: Prettier formatting
- JSON/HTML: Biome format

## CI/CD

GitHub Actions workflows:

- **Lint** (`.github/workflows/lint.yml`): Runs on PRs to main. Builds, lints, and type-checks
- **Release** (`.github/workflows/release.yml`):
  - Triggered on pushes to main with changes in `.changeset/**` or `packages/**`
  - Configured git user: `Lukman Aviccena <lukmanaviccena@gmail.com>`
  - Runs build and lint with `bun run lint:fix && bun run lint`
  - Uses `changesets/action@v1` to create release PRs or publish to npm
  - Requires `NPM_TOKEN` and `GITHUB_TOKEN` secrets

## Commit Convention

Commitlint enforces conventional commits with types: `feat`, `feature`, `fix`, `refactor`, `docs`, `build`, `test`, `ci`, `chore`. Configured in `.commitlintrc.cjs`.

## Changesets Configuration

Changesets is configured in `.changeset/config.json`:

- Changelog generation uses `@changesets/changelog-github` for GitHub releases
- Access level: `public` (packages are published to public npm)
- Internal dependencies are bumped as `patch` versions
- Base branch: `main`
- Workspace protocol-only version bumping enabled

## Publishing

The `scripts/publish.sh` script:

1. Finds all `package.json` files under `packages/`
2. Skips packages with `"private": true`
3. Publishes each package with `bun publish`
4. Creates tags via `changeset tag`

Requires `NPM_TOKEN` environment variable.
