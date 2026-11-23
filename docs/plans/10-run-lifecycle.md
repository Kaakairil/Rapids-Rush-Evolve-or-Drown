# Task 10: Run Lifecycle (start, finish, results)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** Tasks 04–09 · **Produces:** full `RunService` → Tasks 11–14

**Goal:** Press Play → countdown → run → finish/crash → results. The spine that holds everything together.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply. **Review Focus line 5 partially lives here:** steer clamped, out-of-run input ignored.

**Files:**
- Modify: `src/server/Services/RunService.luau` (full lifecycle), `src/server/Main.server.luau` (boot order)
- Test: `src/tests/test_run_lifecycle.luau`

**Interfaces:**
- Consumes: DataService (04), CoinService (05), RaftService (06), HealthService (07–08), ObstacleService (09), `Remotes` (03).
- Produces (docs/functions.md §4.10, docs/architecture.md §9.1):
  - `startRun(player): boolean` — checks profile, allocates raft by party size via `RaftService.getRaftLayout` (solo 1 / duo 1×2 / trio 2+1 / quad 1 big), calls `HealthService.resetForRun`, creates run state, fires `RunState`, starts countdown
  - `endRun(player, reason): RunResult` — reason `"finished" | "outOfLives" | "quit"`; builds `RunResult` (distance, coinsEarned, time, knockoffs, finished, endReason), fires `RunEnded`, adds run coins to profile via `CoinService.add`, destroys raft, returns to Idle; second call is a no-op
  - `handleSteer(player, steer, paddle, brake)` — **clamps steer to [-1, 1]**; ignores players not in a run; rate-limits to `INPUT_RATE_LIMIT` (20/s)
  - `ResetCheckpoint` handler (from Task 09's minimal RunService) and `DiedSignal` subscription retained
  - `checkFinish(player)` — ends run on finish line
  - `update(dt)` — distance tracking, timers for all active runs
- Run state machine: Idle → Countdown → Running → Finished (docs/architecture.md §9.1).

- [ ] **Step 1: Write failing tests** — `handleSteer` with steer 999 stores 1 (clamped); steer -999 stores -1; input from a player not in a run is ignored (no error, no state); `startRun` on a player already in a run returns false; `endRun(p, "finished")` returns a RunResult with `coinsEarned` matching run coins and the profile balance grows by that amount; calling `endRun` twice does not double-add coins
- [ ] **Step 2: Run specs, verify they fail**
- [ ] **Step 3: Implement** full lifecycle (Idle → Countdown → Running → Finished) and boot order in `Main.server.luau`
- [ ] **Step 4: Run specs, verify PASS; Studio playtest**: full 60-second run — start, steer, finish, results, second run works
- [ ] **Step 5: Commit** each file individually (ask approval first)
