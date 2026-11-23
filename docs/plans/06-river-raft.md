# Task 06: River + Raft (steering, current, drag)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** `GameConfig`, `Math` (Task 02) · **Produces:** `RiverService.*`, `RaftService.*` → Tasks 07, 08, 09

**Goal:** A raft that drifts down a straight test river on a custom current, steerable, with knockoff detection.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply (`CURRENT_STRENGTH=40`, `STEER_TORQUE=25`, `DRAG_COEFFICIENT=0.6`, `MAX_SPEED=60`, thresholds). Seat allocation rule lives here.

**Files:**
- Create: `src/shared/Config/RiverConfig.luau`, `src/shared/Config/ObstacleConfig.luau`, `src/server/Services/RiverService.luau`, `src/server/Services/RaftService.luau`
- Test: `src/tests/test_raft_math.luau`

**Interfaces:**
- Consumes: `GameConfig`, `Math.angleBetweenUp` (Task 02).
- Produces (docs/functions.md §4.9 / §4.7):
  - `RiverService.load(riverId: string): Model`, `RiverService.unload()`, `RiverService.getSectionCFrame(index: number): CFrame`, `RiverService.getFinishLineCFrame(): CFrame`, `RiverService.getCheckpoints(): {CFrame}` (exactly `CHECKPOINT_COUNT` = 4), `RiverService.getCurrentDirection(): Vector3`
  - One straight test river: flat channel of parts + terrain water visuals, sections from `RiverConfig`, 4 checkpoint marker CFrames, finish line
  - `RaftService.spawn(player: Player, spawnCFrame: CFrame, partySize: number): Model` — hull Part (light CustomPhysicalProperties) + AlignOrientation (upright) + VectorForce (current) + Seat(s); **seat allocation: partySize 1 → 1 seat; 2 → 1 raft, 2 seats; 3 → 2 rafts (2+1 seats) — caller invokes spawn per raft; 4 → 1 raft, 4 seats** (allocation helper: `RaftService.getRaftLayout(partySize: number): {{seats: number}}` returning the raft split)
  - `RaftService.destroy(player)`, `getRaft(player): Model?`
  - `applyCurrent(raft, direction, strength)`, `applySteer(raft, steer)`, `applyDrag(raft)`
  - `checkKnockoff(raft: Model, impulse: number): (boolean, string)` — `(true, "impact")` above `KNOCKOFF_IMPULSE`, `(true, "flip")` when tilt > `FLIP_THRESHOLD_DEG`, else `(false, "")`
  - `getSpeed(raft)`, `enforceSpeedLimit(raft)`, `resetToCheckpoint(raft, cframe)`, `update(dt)`
- Test river built in code from config — no hand-placed Studio parts yet.

- [ ] **Step 1: Write failing test** — `RaftService.getRaftLayout(1)` = `{{seats=1}}`; `(2)` = `{{seats=2}}`; `(3)` = `{{seats=2},{seats=1}}`; `(4)` = `{{seats=4}}`; `checkKnockoff` with impulse 181 returns `(true, "impact")`, impulse 179 returns `(false, "")`
- [ ] **Step 2: Run spec, verify it fails**
- [ ] **Step 3: Implement** the four modules; physics loop per docs/architecture.md §9.2 (current + drag VectorForce, steering torque per Heartbeat, tilt/impulse knockoff check)
- [ ] **Step 4: Run spec, verify PASS; Studio playtest**: raft drifts forward on current alone, steers with input, stays upright, speed caps at MAX_SPEED
- [ ] **Step 5: Commit** each file individually (ask approval first)
