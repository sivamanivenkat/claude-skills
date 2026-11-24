---
name: repo-restructure
description: Reorganize any code repository (local path or GitHub owner/repo, any language, single-stack or polyglot like Go plus TypeScript) into a clean layered architecture, with optional AI/RAG layers, docs and agent context files. Use this skill whenever the user wants to restructure, reorganize, clean up or tidy a repo's folder structure, fix a messy project layout, apply clean or layered architecture, separate concerns, split a monolith's folders, organize a codebase for AI agents, or asks "where should this code live", even if they never say "restructure".
---

# Repo Restructure

Reorganize a repository into a clean layered architecture, mapped onto each component's own conventions. Plan first, get approval, execute on a branch, verify. The skill is stack-independent: it teaches a method, and Claude works out each stack's conventions from evidence in the repo.

## Layers (stack-independent)

| Layer | Responsibility |
|---|---|
| entry | Bootstrap, server start, DI wiring |
| config | Settings, env, profiles |
| api | Controllers, routes, handlers, request/response DTOs |
| domain | Entities, models, schemas |
| services | Business logic and orchestration |
| adapters | DB repositories, external API clients, queues, cache |
| security | Auth, validation, guards |
| observability | Logging, tracing, metrics, cost tracking |
| data | Migrations, seeds, datasets |
| scripts | Seed, migrate, healthcheck, ops |
| frontend | UI, containerized separately |
| tests | Mirrors the source layout |
| docs | architecture.md, api-reference.md, deployment.md |
| agent context | CLAUDE.md, AGENTS.md, .claude/rules/ |

Not every component needs every layer. Map a layer only where the code exists or the user approves a scaffold.

## Components and conventions

A repo can hold several stacks (for example a Go service next to a TypeScript frontend or worker). Treat each **component**, meaning each directory with its own build manifest, as its own unit:

- Keep each component's top-level folder and restructure inside it. Don't merge stacks into one layer tree; that breaks each toolchain and makes the diff unreviewable.
- For each component, read [references/discovery.md](references/discovery.md) to find its framework conventions, its build/test/lint commands, and the reference types a move can break.
- The framework's own convention wins over the generic layer names. State the layer → folder mapping per component in the plan.
- Cross-component items live at the repo root: shared contracts (OpenAPI, protobuf, shared types), compose files, CI, root docs, and CLAUDE.md/AGENTS.md (with a short section per component).

If the repo uses LLM SDKs, embeddings, vector DBs or prompt files, also read [references/ai-overlay.md](references/ai-overlay.md).

If the user supplies their own target tree, it overrides the mappings.

## Workflow

1. **Inspect.** Clone the repo or use the local path. Stop if the working tree is dirty and ask the user to commit or stash, so the restructure is the only change on the branch. List the components. For each one, find its commands and run build and tests once to record a per-component baseline.
2. **Classify by behavior, not name.** A file called `utils` may be domain logic or an adapter, so use imports, annotations and functions to assign each file a layer. Use a symbol summary (tree-sitter or `npx repomix --compress`) for repos over about 500 files. Flag files that mix layers as split candidates.
3. **Plan, then stop for approval.** List the components and each one's mapping. Show one table per component: current path | target path | action (move/rename/split/keep/delete-candidate/unsure) | reason. List empty layers (scaffold or not), unplaceable files, and risks (dynamic imports, scan roots, path aliases, Docker/CI paths). Move nothing until the plan is approved. Splits, renames and deletions each need explicit approval, because they change more than file locations.
4. **Execute** on branch `restructure/layered-architecture`. Use `git mv` for every move so history follows the files. Fix references as discovery.md describes. Update Dockerfiles, compose files, CI and scripts. Move split code verbatim. Scaffold only approved layers, as stubs with a signature, a doc comment and a TODO. Commit in steps (moves, references, scaffolding, docs) so each step can be reviewed or reverted alone.
5. **Docs and agent context.** Write CLAUDE.md and AGENTS.md (layer map, commands, where new code goes) and `.claude/rules/code-style.md` and `testing.md` derived from the repo's actual tooling. Write `docs/architecture.md` with a Mermaid diagram of the main request flow. Update the README rather than replacing it.
6. **Verify** each component with its own commands and build Docker images if available. Compare against that component's baseline. Fix only breakages the move caused; failures already in the baseline stay and are reported.
7. **Report** using the template below.

## Report template

```
# Restructure report
Components: <name – stack – mapping used>, ...
Moved / split / scaffolded: <counts and notable items per component>
Verification vs baseline: <per component: before → after>
Pre-existing failures (untouched): <list or none>
Unplaced or risky items: <list or none>
Review: git diff main --stat      Undo: delete branch restructure/layered-architecture
```

## Rules

- Work only from an approved plan and never on main. Restructuring touches every file path, so the user needs a chance to correct the mapping and an easy way back.
- Don't change behavior: no logic rewrites, API renames or dependency upgrades. This keeps the baseline tests a valid check that the move was safe.
- Leave secrets alone. Don't print or move them. Keep `.env` in place and add an example file if missing, so nothing sensitive lands in the diff or the report.
- If verification can't pass, stop and report instead of forcing fixes. Forced fixes hide the real cause and mix behavior changes into a structural diff.
