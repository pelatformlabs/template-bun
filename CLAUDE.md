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

**Important**: This project uses Bun as the package manager. Always use `bun` commands instead of `npm` or `yarn`. The `packageManager` field is set to `bun@1.3.6` in `package.json`.

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
├── core/          # Core package (template-ready)
└── main/          # Main package (template-ready)

apps/              # Optional applications consuming packages
├── web/           # Web application (template-ready)
└── docs/          # Documentation (template-ready)

examples/          # Example implementations
├── next/          # Next.js example (template-ready)
└── vite/          # Vite example (template-ready)
```

Workspaces are defined in both `package.json` and `bunfig.toml`. The workspace patterns are `packages/**`, `apps/**`, and `examples/**`:

- **packages/**: Published or internal libraries
- **apps/**: Applications that consume packages (docs, demos)
- **examples/**: Example implementations showcasing usage

**Note**: All workspace directories are currently template-ready (empty with .gitkeep files). Populate them according to your project needs.

## Turborepo Pipeline

The `turbo.json` defines task dependencies:

- `build`: Depends on `^build` (build dependencies first). Outputs: `.next/**`, `.source/**`, `build/**`, `dist/**`, `out/**`
- `dev`: Persistent, uncached
- `start`: Depends on `^build`, persistent
- `lint`: No dependencies
- `types:check`: Depends on `^build`
- `clean`/`clean:all`: Uncached, non-persistent

The `globalEnv` includes `NODE_ENV` for environment-aware caching.

## Code Quality

**Biome** (`biome.jsonc`) extends `@pelatform/biome-config/base` for consistent linting and formatting across the project. Lint-staged with Husky pre-commit hooks ensures code quality before commits.

**Lint-staged configuration** (run automatically on pre-commit):

- TypeScript/JavaScript files: `biome check --write --no-errors-on-unmatched`
- Markdown/YAML files: `prettier --write`
- JSON/HTML files: `biome format --write --no-errors-on-unmatched`

**Note**: Husky hooks are installed via `bun install` (triggers the `prepare` script).

## CI/CD

GitHub Actions workflows:

- **Lint** (`.github/workflows/lint.yml`):
  - Runs on pull requests to `main` branch
  - Installs dependencies with frozen lockfile
  - Executes: `bun run build`, `bun run lint:fix && bun run lint`, `bun run types:check`

- **Release** (`.github/workflows/release.yml`):
  - Triggered on pushes to `main` with changes in `.changeset/**` or `packages/**`
  - Git user: `Lukman Aviccena <lukmanaviccena@gmail.com>`
  - Executes: `bun run build`, `bun run lint:fix && bun run lint`
  - Uses `changesets/action@v1` to create version PRs or publish to npm
  - Required secrets: `NPM_TOKEN`, `GITHUB_TOKEN` or `GH_PAT`

## Commit Convention

Commitlint enforces conventional commits with types: `feat`, `feature`, `fix`, `refactor`, `docs`, `build`, `test`, `ci`, `chore`. Configured in `.commitlintrc.cjs`.

## Changesets Configuration

Changesets is configured in `.changeset/config.json` for the `pelatformlabs/template` repository:

- Changelog generation uses `@changesets/changelog-github` for GitHub releases
- Access level: `public` (packages are published to public npm)
- Internal dependencies are bumped as `patch` versions
- Base branch: `main`
- Workspace protocol-only version bumping enabled (`bumpVersionsWithWorkspaceProtocolOnly`)

## Publishing

The `scripts/publish.sh` script:

1. Finds all `package.json` files under `packages/` (excluding `node_modules`)
2. Skips packages with `"private": true`
3. Publishes each package with `bun publish` (continues on error)
4. Creates tags via `changeset tag`

Requires `NPM_TOKEN` environment variable for npm authentication.

## Package Development Guidelines

### Creating a New Package

When adding packages to the monorepo, follow this structure:

```
packages/your-package/
├── src/
│   ├── index.ts          # Main entry point and exports
│   └── ...               # Other source files
├── package.json          # Package metadata
├── tsconfig.json         # TypeScript configuration
├── tsup.config.ts        # Build configuration (if using tsup)
└── README.md             # Package documentation
```

Key points:

- Each package should have its own `package.json` with proper `name`, `version`, and `exports` field
- Use workspace protocol for internal dependencies: `"@pelatform/other-package": "workspace:*"`
- Define build scripts if the package needs compilation
- Ensure proper TypeScript exports configuration in `tsconfig.json`

### Testing

When adding tests to packages, use the workspace protocol for test dependencies and ensure test scripts are defined in each package's `package.json`. Run tests with:

```bash
# Run all tests
bun run test

# Run tests for a specific package
bun run test --filter=@pelatform/your-package
```

### Example Apps

The `examples/` directory showcases implementations:

- **next/**: Next.js integration examples
- **vite/**: Vite/Rollup integration examples

These are references for users consuming your packages.

## Environment Variables

Required for CI/CD and publishing:

- `NPM_TOKEN`: npm authentication token for publishing packages
- `GITHUB_TOKEN` or `GH_PAT`: GitHub token for creating releases via Changesets action

## Common Issues

### Turbo Cache Issues

If you encounter stale cache issues with Turborepo:

```bash
bun run clean          # Remove build artifacts
bun run clean:all      # Deep clean including node_modules and locks
```

### Husky Hooks Not Working

If pre-commit hooks aren't running:

```bash
bun run prepare        # Reinstall Husky hooks
```

### Workspace Dependencies

When adding dependencies to workspaces, use the workspace protocol (`workspace:*`) for internal dependencies in `package.json`:

```json
{
  "dependencies": {
    "@pelatform/other-package": "workspace:*"
  }
}
```

To add packages via CLI:

```bash
# Add to a specific workspace
bun add <package> --filter=@pelatform/your-package

# Add to root (dev dependency)
bun add <package> -D
```
