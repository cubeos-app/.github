# Contributing to CubeOS

Thank you for your interest in contributing to CubeOS! This guide covers everything you need to get started.

## Development Setup

- **Go 1.24+** (CGO disabled — all builds use `CGO_ENABLED=0`)
- **Node.js 20+** (for the dashboard)
- **Docker** with Swarm mode enabled (`docker swarm init`)
- **shellcheck** (for shell script linting)

## Repository Guide

| Repo | Language | What lives here |
|------|----------|-----------------|
| `api` | Go | Backend REST API — handlers, managers, orchestrator, FlowEngine |
| `hal` | Go | Hardware Abstraction Layer — GPIO, I2C, sensors, network, power |
| `dashboard` | Vue 3 | Web management UI — Pinia stores, Tailwind styling |
| `coreapps` | Docker Compose | System service configs (Pi-hole, NPM, registry, etc.) |
| `releases` | Packer + Shell | Image builds, installer scripts, release automation |
| `docs` | Markdown | Architecture docs, user guides, API contracts |
| `demo` | CI config | Builds and deploys demo.cubeos.app |

## Code Style

### Go (api, hal)

- Run `gofmt` before committing — CI enforces this
- All exported HTTP handlers **must** include complete Swagger annotations:
  `@Summary`, `@Description`, `@Tags`, `@Param`, `@Success`, `@Failure`, `@Router`
- Handle all errors — never use `_` to discard an error return
- No hardcoded IPs, ports, or host URLs — use `os.Getenv()` with defaults
- No CGO — use `modernc.org/sqlite`, not `mattn/go-sqlite3`
- Use `context.Context` for cancellation and timeouts
- Prefer table-driven tests

### Vue / TypeScript (dashboard)

- Composition API with `<script setup>` — never Options API
- Pinia for state management
- Tailwind CSS utility classes only — no custom CSS files
- Monochrome SVG icons only — **no emojis** anywhere in the UI
- Must be fully responsive (PC and smartphone)

### Shell Scripts (releases, coreapps)

- `shellcheck` must pass with no warnings
- Use `set -euo pipefail` at the top of every script
- Quote all variables

## Pull Request Workflow

1. **Fork** the repository
2. Create a **feature branch**: `feat/description`, `fix/description`, or `docs/description`
3. Make your changes and ensure tests pass
4. Submit a **PR against `main`**
5. Wait for CI to pass and a maintainer review

Keep PRs focused — one feature or fix per PR.

## Testing

| Repo | Command |
|------|---------|
| api | `cd api && go test -v ./...` |
| hal | `cd hal && go test -v ./...` |
| dashboard | `cd dashboard && npm run test` |
| releases | `bash -n scripts/*.sh` (syntax check) + `shellcheck scripts/*.sh` |

Run the full test suite before submitting a PR.

## Architecture

For architecture decisions, data flows, and design rationale, see the [docs repo](https://github.com/cubeos-app/docs):

- `02_ARCHITECTURE.md` — component interaction and data flows
- `03_DATABASE_SCHEMA.md` — full SQL schema
- `04_BOOT_SEQUENCE.md` — startup order and recovery
- `07_API_CONTRACTS.md` — OpenAPI endpoint specifications

## License

CubeOS is licensed under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0). By contributing, you agree that your contributions will be licensed under the same terms.
