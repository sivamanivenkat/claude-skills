# Discovery Guide

How to work out an unfamiliar component's conventions, reference types and verify commands from evidence in the repo. Use it for every component, whatever the language.

## Contents
1. Finding components
2. Finding the framework's conventions
3. Finding build, test and lint commands
4. Finding what a move can break
5. Known pitfalls (examples, not a complete list)

## 1. Finding components

A component is a directory with its own build manifest. Search for these (skip `node_modules`, `vendor`, `dist`, `build`, `target`, `.venv`):

| Manifest | Ecosystem |
|---|---|
| pyproject.toml, requirements.txt, setup.py | Python |
| pom.xml, build.gradle(.kts) | JVM |
| package.json | JS/TS (backend, frontend or tooling: check dependencies) |
| go.mod | Go |
| *.csproj, *.sln | .NET |
| Cargo.toml | Rust |
| Gemfile | Ruby |
| composer.json | PHP |
| mix.exs, pubspec.yaml, Package.swift, CMakeLists.txt | Elixir, Dart/Flutter, Swift, C/C++ |

Workspace roots (a `package.json` with `workspaces`, `go.work`, Cargo `[workspace]`, a Gradle multi-project build, a `.sln`) group components. Treat each member as its own component.

To tell a frontend from a backend in `package.json`, look at dependencies: UI frameworks and a bundler mean frontend; an HTTP server framework means backend; neither means shared library or tooling.

## 2. Finding the framework's conventions

Gather evidence in this order and stop when the picture is clear:
1. **The manifest's dependencies** name the framework (Spring Boot, FastAPI, Gin, NestJS, Rails, Laravel, Axum).
2. **The existing layout.** Many repos are already half-conventional. Keep what is already right and move only what is wrong.
3. **The framework's documented structure.** If unsure, check the framework's own docs or starter template rather than guessing.
4. **Feature-based vs layer-based.** Some frameworks organize by feature (NestJS modules, Django apps, Rails engines). Apply the layers inside each feature and keep the feature folders.

The framework's convention wins over the generic layer names. State the mapping (layer → folder) in the plan so the user can correct it.

## 3. Finding build, test and lint commands

Prefer the repo's own commands over defaults, in this order:
1. Scripts in the manifest (`package.json` scripts, Makefile targets, `justfile`, `taskfile`).
2. CI config (`.github/workflows`, `.gitlab-ci.yml`, `Jenkinsfile`): what CI runs is the truest verify command.
3. README or contributing docs.
4. Ecosystem defaults: `pytest`, `mvn verify` / `./gradlew build`, `go build ./... && go test ./...`, `npm test && npm run build`, `dotnet build && dotnet test`, `cargo build && cargo test`, `bundle exec rake test`.

Run each component's commands once before any change and record the result as that component's baseline. A failure that already exists in the baseline is not a regression; report it and do not fix it.

## 4. Finding what a move can break

For each moved file, search the repo for its old path, its module, package or namespace name, and its exported symbols. Check these reference types for the stack:

- **Imports and package or namespace declarations**, which often must match the directory.
- **Path aliases and module maps** in config (tsconfig paths, bundler aliases, test-runner module maps, Go module path, Python package discovery).
- **Entry points** named in config: start scripts, `main` fields, server module paths, application class names.
- **Scan roots and auto-discovery:** frameworks that find components by scanning a package or folder.
- **Generated code and codegen config:** annotation processors, protobuf/OpenAPI generators, ORM migration locations.
- **Resource and asset paths** loaded at runtime (templates, prompts, SQL, config files).
- **Build and packaging files:** Dockerfiles, compose files, CI workflows, Makefiles, deployment manifests.
- **Dynamic references:** reflection, dynamic imports, string-built paths. These can't be found by static search, so list each one as a risk in the plan.

Prefer language-aware tooling (IDE refactor engine, ts-morph, gopls, rope, OpenRewrite) over regex for rewriting imports, since it follows symbols and not text.

## 5. Known pitfalls (examples, not a complete list)

- **Spring Boot:** `@SpringBootApplication` scans its own package and below; `@ComponentScan`, `@EntityScan` and `@EnableJpaRepositories` can name other packages.
- **Go:** code moved under `internal/` can no longer be imported by other modules; flag it if another module depends on it.
- **Python:** a new directory needs `__init__.py` where the packaging expects one; check Alembic `script_location`, uvicorn/gunicorn module paths and pytest `pythonpath`.
- **TypeScript:** barrel `index.ts` files and path aliases hide imports; Angular also has `angular.json` asset and style paths.
- **.NET:** namespaces follow folders by convention; `<ProjectReference>` entries and the `.sln` must be updated together.
- **Django, Rails, Laravel:** keep the app/engine/module-per-feature layout; apply layers inside each unit.

When working in a stack not listed here, look for its equivalent of each item: what auto-discovers code, what must match the directory, and what config names a file path.
