# Task 14: Persistence Round-Trip + Finish Screen

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** DataService (04), RunService (10), EvolutionService (13) · **Produces:** save round-trip, finish screen, Evolve menu → Task 15 (final playtest)

**Goal:** What you earn, you keep. Coins, evolution, and best distance survive a leave/rejoin — and the run ends on a proper results screen.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply.

**Files:**
- Create: `src/server/Services/ProgressionService.luau`, `src/client/Controllers/MenuController.luau` (minimal Evolve menu)
- Modify: `src/server/Services/RunService.luau` (record on end), `src/client/Controllers/HUDController.luau` (finish screen)
- Test: `src/tests/test_persistence.luau`

**Interfaces:**
- Consumes: `DataService` (04), `RunService.endRun` (10), `EvolutionService` (13), `RunEnded`/`DataLoaded` remotes (03). MenuController enables its buttons on `DataLoaded`.
- Produces (docs/functions.md §4.3):
  - `ProgressionService.getBestDistance(player, riverId): number`; `recordRun(player, result: RunResult): boolean` — true only on a new best; `recordCrash(player, force, kind)`; `getStats(player)`
  - `endRun` now records distance via ProgressionService before firing `RunEnded`
  - Finish screen (HUD): distance, coins, time, best crash, Play Again button
  - Minimal Evolve menu (MenuController): grid listing Frog (silhouette + cost when locked), Buy/Equip buttons calling the RemoteFunctions, balance display
- Persistence invariant (PRD FR-6): coins, owned evolutions, equipped evolution, best distance survive rejoin.

- [ ] **Step 1: Write failing test** — profile with 300 coins buys frog, records best distance 5000, records a crash; simulate re-load; all persist; `recordRun` with distance 4000 returns false (no new best), with 6000 returns true
- [ ] **Step 2: Run spec, verify it fails**
- [ ] **Step 3: Implement** ProgressionService, run-end recording, finish screen, minimal Evolve menu
- [ ] **Step 4: Run spec, verify PASS; Studio test**: play a run, leave, rejoin — coins, Frog, best distance all still there; finish screen populates; Evolve menu shows owned/locked state correctly
- [ ] **Step 5: Commit** each file individually (ask approval first)
