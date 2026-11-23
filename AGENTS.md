# AGENTS.md — Rapids Rush: Evolve or Drown Project Guide

## Project purpose

Rapids Rush: Evolve or Drown is a Roblox physics-based river racer: players ride rafts down a chaotic river, take ragdoll hits (HP/stamina), revive teammates, collect coins, and spend them on creature evolutions. Built in Roblox Studio with Luau. This is the owner's first Roblox project, built solo; the goal is a published experience that friends and other players can enjoy — not a monetization-first product. Do not add live monetization, seasonal rivers, or race lobbies before the core loop is fun and the PRD's P0 features are done.

`PRD.md` is the source of truth for scope, functional requirements, and deliverables; `design.md` holds the game design decisions, `docs/architecture.md` the technical runtime view, `docs/functions.md` the module/function reference, and `docs/DECISIONS.md` the decision checkpoint. When this guide conflicts with the PRD, follow the PRD and update this guide if needed. Keep recommendations and reports evidence-based; state limitations and inconclusive results clearly. Every claimed behavior must trace to a test, a verification command, or a Studio playtest.

## Repository and environment

- Roblox Studio with **Rojo** for file-based source control (`default.project.json` maps `src/` into Studio); Luau with `--!strict` mode.
- Server code lives in `src/server/` (Main.server.luau, Services/); client controllers in `src/client/Controllers/`; shared config, remotes, types, and utilities in `src/shared/` (Config/, Net/, Util/). Tests live in `src/tests/` (TestEZ). Planning documents (`PRD.md`, `design.md`, `docs/*.md`, `docs/plans/`) live at the workspace root.
- Dependencies: ProfileStore (data saving) and TestEZ (tests) installed as vendored packages or via Wally; do not add a dependency when the Roblox API or an installed module covers the need.
- Run workflow from the repository root:

```powershell
rojo serve            # sync src/ into a running Roblox Studio session
rojo build -o RapidsRush.rbxlx   # build a place file (no Studio session)
# Tests: run TestEZ spec via Studio command bar or a test-place build
```

For focused work, run the relevant test spec or Studio playtest directly. Do not run a full publish pipeline just to validate an unrelated code change.

## Standards for this project

- **Server-authoritative, no silent failures.** The server owns coins, HP/stamina, evolutions, run results, and race placement. The client only sends input, never results. A forged, malformed, or out-of-run remote payload must be clamped, rate-limited, or ignored — never trusted.
- **Correctness discipline.** Coin spends validate balance before deducting; run-end coin totals are added in one transaction; data saves go through ProfileStore session locking with auto-save every 60 seconds and on leave/shutdown. These invariants are pinned in docs/architecture.md §9 and verified by tests.
- **Typed code and config-driven tuning.** All public module functions carry Luau type annotations; every tunable number (damage, stamina drain, forces, costs) lives in a config file in `src/shared/Config/`, never hard-coded in scripts.
- No real-world violence or gory content; ragdoll crashes are cartoonish. No pay-to-win: paid items add convenience or cosmetics only.
- Tests are deterministic with no network access in unit tests; pure logic (coin math, costs, cooldowns, migrations, clamps) is covered by TestEZ specs, physics feel by manual Studio playtests.

## Data handling

- Player data is saved through ProfileStore (session-locked wrapper around DataStoreService); leaderboards use OrderedDataStore with integer scores (distance in centimeters).
- Schema changes increment the profile `version` field and add a migration in DataService. Never remove a field without a migration; never modify an already-released migration.
- Remote names are defined once in `src/shared/Net/Remotes.luau`; no string-typed remote names outside that file.
- Keep generated artifacts (`*.rbxlx` builds, `sourcemap.json`, caches) out of Git per `.gitignore`. Do not commit credentials, API keys, or place files with embedded secrets.

## Development workflow

1. Read the relevant PRD sections, `design.md`, existing implementation, configuration, and tests before changing behavior. Check repository status and preserve unrelated or in-progress changes.
2. When asked to check or recheck the game, audit implemented behavior against the PRD and design acceptance criteria. Identify concrete gaps, mismatches, and missing tests; distinguish confirmed defects from unimplemented future scope. Do not stop at a superficial status report when the request asks to fix or continue the work.
3. For multi-file or behaviorally significant work, write a focused implementation plan in `docs/plans/` (gapless, phased, one commit per task), identify affected modules and acceptance checks, and review the plan against the PRD before editing. Keep small fixes direct.
4. Work in dependency order: shared config and types, server services (data → coins → health/raft/obstacles → run lifecycle), then client controllers (input → camera → HUD → menus → effects). Keep changes scoped and follow existing module patterns.
5. For each planned unit: write the failing test first, run it to confirm it fails, implement the minimal code, run it to confirm it passes. Physics-feel work that cannot be unit-tested gets a manual Studio playtest checklist instead.
6. Add or update focused tests alongside behavior changes. Run the narrow spec first, then relevant broader specs.
7. Inspect the final diff for scope, correctness, generated files, and accidental secret inclusion. Report checks run and remaining gaps clearly.
8. Commits: when ready to commit, commit and push **each file individually with one comment per commit**. Never batch multiple files into a single commit. Push to `main` only. Never add any other author or contributor — no Co-Authored-By trailers, no credit to Claude or anyone else; commits are credited to the repository owner (Khairil-12) only. Ask approval before every commit and every push.

## Collaboration preferences

- The user commonly asks for a review against the PRD, including a gap check, before moving to the next task. Treat this as a request for evidence-based verification, not just a summary of completed files.
- The user prefers work to proceed task by task, with relevant implementation and verification completed before advancing.
- The user asks to check before and after each implementation. Check state before starting a task and verify the result after finishing it.
- Keep communication direct and concise. When prompted to audit, lead with concrete gaps and their impact; when prompted to implement, make the change and report verification results.
- Interpret short follow-up prompts in the context of the current project and prior task, while respecting any explicit instruction to pause, avoid tools, or only provide text.

## Code and tests

- Prefer small, typed functions and Roblox API or already-installed modules. Avoid unrelated refactors and speculative abstractions.
- Add Luau type annotations and concise comments to public functions. Handle invalid external inputs (remote payloads, config values) explicitly; do not add defensive branches for impossible internal states.
- Use TestEZ for tests. Add focused regression tests for behavior changes, especially coin spend validation, damage/ragdoll thresholds, revive eligibility, checkpoint rewards, and input clamping.
- Keep tests deterministic. Avoid network access in unit tests; mock remote/data layers and use small fixtures.
- Run relevant TestEZ specs after code changes and verify physics feel in Studio playtests. Report known platform limitations rather than hiding them.
- Do not claim the game works unless it was actually run in Studio.

## Completion standard

Before finishing, inspect the diff, run focused TestEZ specs and Studio playtests for changed behavior, and state any checks not run. Commit and push each file individually with one comment per commit, with approval before each. The repository is remote-linked to the owner's GitHub (`Khairil-12`); publish to Roblox only when the PRD milestone exit criteria are met.
