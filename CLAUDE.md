# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Bun monorepo template for building and publishing TypeScript/Node packages. It uses:
- **Bun** as the package manager (required: `bun >= 1.0.0`)
- **Turborepo** for task orchestration across workspaces
- **Biome** for linting and formatting (extends `@pelatform/biome-config/base`)
- **Changesets** for versioning and publishing
- **Husky** for Git hooks
- **Commitlint** with `@commitlint/config-conventional` for commit message linting

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

## Workspace Structure

```
packages/           # Published or internal packages
├── core/          # Core package
└── main/          # Main package

apps/              # Optional applications consuming packages
├── web/           # Web application
└── docs/          # Documentation
```

Workspaces are defined in both `package.json` and `bunfig.toml`.

## Turborepo Pipeline

The `turbo.json` defines task dependencies:
- `build`: Depends on `^build` (build dependencies first). Outputs: `.next/**`, `.source/**`, `build/**`, `dist/**`, `out/**`
- `dev`: Persistent, uncached
- `start`: Depends on `^build`, persistent
- `lint`: No dependencies
- `types:check`: Depends on `^build`
- `clean`/`clean:all`: Uncached, non-persistent

## CI/CD

GitHub Actions workflows:
- **Lint** (`.github/workflows/lint.yml`): Runs on PRs to main. Builds, lints, and type-checks
- **Release** (`.github/workflows/release.yml`): Triggered on pushes to main with changes in `.changeset/**` or `packages/**`. Runs build/lint then creates release PR or publishes to npm

## Commit Convention

Commitlint enforces conventional commits with types: `feat`, `feature`, `fix`, `refactor`, `docs`, `build`, `test`, `ci`, `chore`.

## Publishing

The `scripts/publish.sh` script:
1. Finds all `package.json` files under `packages/`
2. Skips packages with `"private": true`
3. Publishes each package with `bun publish`
4. Creates tags via `changeset tag`

Requires `NPM_TOKEN` environment variable.
