# Rapids Rush — Phase 1 Build Plans: Overview

**One plan per task.** Review them in order; each plan states exactly what it consumes from the previous one and what it produces for the next, so gaps show up as a missing handoff.

**Spec:** [PRD.md](../../PRD.md), [design.md](../../design.md), [docs/architecture.md](../architecture.md), [docs/functions.md](../functions.md) — implements PRD v0.2 P0 features (F01–F10, F22–F26).

## Global Constraints (apply to every task)

- Luau `--!strict` in every module.
- All tunable numbers live in `src/shared/Config/`, never hard-coded. Starting values (verbatim from docs/functions.md §1.1): `MAX_HP=100`, `MAX_STAMINA=100`, `LIGHT_DAMAGE=25`, `HEAVY_DAMAGE=50`, `RAGDOLL_LIGHT_S=1`, `RAGDOLL_HEAVY_S=2`, `REVIVE_HP_PERCENT=25`, `REVIVE_HOLD_S=3`, `BLEED_OUT_S=20`, `SELF_REVIVE_HP_PERCENT=25`, `STAMINA_DRAIN_RAFT=8`, `STAMINA_DRAIN_WATER=25`, `STAMINA_REGEN=15`, `CURRENT_STRENGTH=40`, `STEER_TORQUE=25`, `DRAG_COEFFICIENT=0.6`, `MAX_SPEED=60`, `FLIP_THRESHOLD_DEG=60`, `KNOCKOFF_IMPULSE=180`, `RESPAWN_DELAY=2`, `CHECKPOINT_REWARD=10`, `CHECKPOINT_COUNT=4`, `STARTING_LIVES=3`, `MAX_LIVES=5`.
- Server-authoritative: the client sends input only; every reward, damage number, and run result is computed on the server.
- Remote names are defined once in `src/shared/Net/Remotes.luau`; no string remote names elsewhere.
- Player data through ProfileStore (store key `"RapidsRush_v1"`), schema `version: 1`, auto-save 60 s + on leave/shutdown.
- Party-size raft allocation (one rule): 1 player → 1 raft; 2 → 1 raft, 2 seats; 3 → 2 rafts (2+1); 4 → 1 big raft, 4 seats.
- Co-op core: revive to 20–25% HP; solo gets one self-revive per run.
- Commits: one file per commit, ask approval before every commit and push; author Khairil-12 only, never Co-Authored-By.

## Review Focus (the gaps most likely to bite)

1. Stamina drains use the wrong table for context — on-raft paddle/dodge = `STAMINA_DRAIN_RAFT` (8/s); in-water swim/double-jump = `STAMINA_DRAIN_WATER` (25/s). → Task 07
2. Ragdoll flinch on zero-damage grazes or wrong duration — flinch only when damage > 0, duration `RAGDOLL_LIGHT_S`/`RAGDOLL_HEAVY_S`. → Task 07
3. Revive from invalid state (target not downed, too far, already downed reviver) must return false with no side effects. → Task 08
4. Checkpoint reward farming by re-crossing — reward only the first time an index is reached. → Task 09
5. Unclamped remote payloads (steer 999, negative spend) — steer clamped to [-1, 1], spend rejects amount ≤ 0. → Tasks 05, 10

## Task Chain (handoffs)

| # | Plan | Consumes (from previous) | Produces (for next) |
|---|------|--------------------------|---------------------|
| 01 | [Rojo scaffold](01-rojo-scaffold.md) | — | Buildable `default.project.json`, folder tree |
| 02 | [Types, Math, Config](02-types-math-config.md) | Folder tree | `Math`, `GameConfig`, `EvolutionConfig`, `Types` |
| 03 | [Remotes and Signal](03-remotes-signal.md) | Folder tree | `Remotes.get(name)` (incl. `DataLoaded`), `Signal` |
| 04 | [DataService](04-dataservice.md) | `Types` (02) | `DataService.*`, `Main.server` boot |
| 05 | [CoinService](05-coinservice.md) | `DataService` (04) | `CoinService.*` |
| 06 | [River + Raft](06-river-raft.md) | `GameConfig` (02) | `RiverService.*`, `RaftService.*` |
| 07 | [HealthService](07-healthservice.md) | `GameConfig` (02), `DamageEvent` (03), `RaftService` (06) | HP/stamina/ragdoll/downed, `setInWater` (own water check), `DiedSignal` (breaks 07↔09/10 cycle) |
| 08 | [Revive wiring](08-revive.md) | HealthService (07), rafts (06) | Complete revive/swim-back/second-wind loop |
| 09 | [Obstacles + checkpoints](09-obstacles-checkpoints.md) | `HealthService.damage` (07), `checkKnockoff` (06) | `ObstacleService.*`, minimal `RunService` (incl. `ResetCheckpoint` handler + `DiedSignal` sub), checkpoints |
| 10 | [Run lifecycle](10-run-lifecycle.md) | Tasks 04–09 | Full `RunService` start→finish |
| 11 | [Client input + camera](11-client-input-camera.md) | `Remotes` (03) | `InputController`, `CameraController` |
| 12 | [HUD + effects](12-hud-effects.md) | All S→C remotes (03) | `HUDController`, `EffectsController` |
| 13 | [Frog evolution + coin pickups](13-frog-evolution.md) | `CoinService` (05) | `EvolutionService.*`, `CoinPickupService` |
| 14 | [Persistence + finish screen](14-persistence-finish.md) | 04, 10, 13 | Save round-trip, finish screen, Evolve menu |
| 15 | [Party setup + publish](15-party-publish.md) | All | Playable published experience |

## Change Log

| Date | Change |
|------|--------|
| 2026-11-23 | Created overview: 15 per-task plans with handoff table |
| 2026-11-23 | Gap review: added `DataLoaded` (03/11/14), `setInWater` water check (07), `ResetCheckpoint` handler + `DiedSignal` circular-dep resolution (07/09/10), fixed Task 14 MenuController Files block |
