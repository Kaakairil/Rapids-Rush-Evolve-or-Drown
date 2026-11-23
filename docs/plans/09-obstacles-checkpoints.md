# Task 09: Obstacles, Damage Routing, Checkpoints

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** `HealthService.damage` (07), `RaftService.checkKnockoff` (06), `ObstacleConfig` (06), `KnockoffEvent` (03) · **Produces:** `ObstacleService.*`, minimal `RunService`, checkpoints → Task 10

**Goal:** Logs and rocks that actually hurt you, and checkpoint flags that save progress.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply. **Review Focus line 4 lives here:** checkpoint reward only on first crossing.

**Files:**
- Create: `src/server/Services/ObstacleService.luau`, `src/server/Services/RunService.luau` (minimal — full lifecycle in Task 10)
- Test: `src/tests/test_checkpoint.luau`, `src/tests/test_obstacle_damage.luau`

**Interfaces:**
- Consumes: `ObstacleConfig` (Task 06), `HealthService.damage/down/die` (Task 07), `RaftService.checkKnockoff/resetToCheckpoint` (Task 06), `Remotes` (Task 03), `CoinService.add` (Task 05).
- Produces:
  - `ObstacleService.init()` — pools (40/type) from `ObstacleConfig`
  - `ObstacleService.spawnSection(riverId, sectionIndex, startZ): {Instance}`, `despawnSection(sectionIndex)`
  - `ObstacleService.update(dt, raftPosition)` — drift logs, active window = current + next section
  - `ObstacleService.onHit(obstacle, raft, impulse)` — routes damage: log/small rock → `HealthService.damage(player, LIGHT_DAMAGE, false)`; barrel/large rock → `damage(player, HEAVY_DAMAGE, true)`; above `KNOCKOFF_IMPULSE` → knockoff (player off raft) + `KnockoffEvent(force, kind)`
  - `ObstacleService.getKind(obstacle): string`
  - Minimal `RunService` (docs/functions.md §4.10 subset): `startRun/getRunState/loseLife/respawn/crossCheckpoint/addCoins/update/isInRun`, **plus the `ResetCheckpoint` remote handler** (validates player in run → `HealthService.respawnAtCheckpoint`) and a `DiedSignal` subscription from HealthService (Task 07 circular-dependency note) that routes to `loseLife`
  - `crossCheckpoint(player, index)` — saves `checkpointIndex` (monotonic), grants `CHECKPOINT_REWARD` **only for a new index**, fires `RunState`
  - Checkpoint trigger zones at the 4 CFrames from `RiverService.getCheckpoints()`
  - Death respawn: `HealthService.respawnAtCheckpoint` — full HP/stamina, last checkpoint CFrame

- [ ] **Step 1: Write failing tests** — `onHit` log impulse above threshold calls `damage(p, 25, false)`; barrel calls `damage(p, 50, true)`; `crossCheckpoint(1)` grants 10 coins and sets `checkpointIndex=1`; second `crossCheckpoint(1)` grants 0 coins (Review Focus line 4); `crossCheckpoint(3)` after `crossCheckpoint(1)` works (monotonic forward); `loseLife` at 0 lives ends run with reason `"outOfLives"`
- [ ] **Step 2: Run specs, verify they fail**
- [ ] **Step 3: Implement** obstacle templates + pooled spawning, impulse → damage routing, checkpoint zones, minimal RunService
- [ ] **Step 4: Run specs, verify PASS; Studio playtest**: log hit = 25 HP + short flinch; barrel = 50 HP + long ragdoll; checkpoint flag saves + pays once; dying respawns at last checkpoint with full HP
- [ ] **Step 5: Commit** each file individually (ask approval first)
