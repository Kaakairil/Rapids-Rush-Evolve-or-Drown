# Task 07: HealthService (HP, stamina, ragdoll, downed)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** `GameConfig` (02), `DamageEvent` (03), `RaftService.getRaft` (06) · **Produces:** full health state API → Tasks 08, 09

**Goal:** The survival layer: HP damage with ragdoll flinch on every hit, context-sensitive stamina, downed state with bleed-out.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply. **Review Focus lines 1–2 live here:** wrong stamina drain table for context; ragdoll on zero-damage grazes.

**Files:**
- Create: `src/server/Services/HealthService.luau`
- Test: `src/tests/test_health_service.luau`

**Interfaces:**
- Consumes: `GameConfig` (Task 02), `Remotes.get("DamageEvent")` (Task 03), `RaftService.getRaft` (Task 06).
- Produces (docs/functions.md §4.6):
  - `HealthService.resetForRun(player)` — HP/stamina to max, `downed=false`, `inWater=false`, `selfReviveUsed=false`, `checkpointIndex=0`
  - `HealthService.getHp(player): number`, `getStamina(player): number`
  - `HealthService.damage(player, amount: number, heavy: boolean)` — HP loss; **ragdoll flinch only when `amount > 0`**, duration `heavy and RAGDOLL_HEAVY_S or RAGDOLL_LIGHT_S`; fires `DamageEvent(amount, position, ragdollDuration)`; calls `down()` at HP ≤ 0
  - `HealthService.heal(player, amount)`
  - `HealthService.drainStamina(player, amount)`, `regenStamina(player, amount)`
  - `HealthService.setInWater(player, inWater: boolean)` — flips the drain-rate context; **fired by the server when the character exits all raft seats / touches water terrain (checked in `update(dt)` — position vs river bounds + seat occupancy; no separate service)**
  - `HealthService.down(player)` — `downed=true`, bleed-out timer `BLEED_OUT_S`, fires `DownedEvent(player, bleedOutTime)`
  - `HealthService.revive(reviver, downedPlayer): boolean` — validated in Task 08
  - `HealthService.swimBack(player): boolean` — fails at stamina ≤ 0
  - `HealthService.selfRevive(player): boolean` — once per run, restores `SELF_REVIVE_HP_PERCENT`
  - `HealthService.die(player, kind)` — fires `DiedSignal(player, kind)`; RunService subscribes (see circular dependency note)
  - `HealthService.respawnAtCheckpoint(player)` — full HP/stamina restore at checkpoint CFrame
  - `HealthService.update(dt)` — bleed-out countdown, stamina regen when drifting
- Per-run state kept server-side: `hp, stamina, downed, inWater, selfReviveUsed, checkpointIndex`.
- Water detection (what calls `setInWater`): in `update(dt)`, server checks each character — seated on a raft seat (`Seat.Occupant`) → not in water; otherwise position below river surface / inside river bounds → in water. No separate service; HealthService owns it.
- Ragdoll: server-side — Humanoid state change + BallSocket constraints; duration scaled by hit force.
- **Circular dependency note:** `HealthService.die` must call `RunService.loseLife`, but RunService (Task 09/10) depends on HealthService. Resolution: HealthService fires a `DiedSignal` (from Task 03's Signal) instead of calling RunService directly; RunService subscribes on start. No direct call, no loop.

- [ ] **Step 1: Write failing tests** — `damage(30, false)` → HP 70 and flinch duration 1; `damage(50, true)` → flinch duration 2; four light hits (25×4) → `downed=true` and `DownedEvent` fired; with `inWater=true`, draining 50 stamina over 2 s (`update(dt)` ticks) loses at the 25/s rate, not 8/s (Review Focus line 1); `damage(0, false)` does NOT ragdoll (Review Focus line 2); `selfRevive` returns true once then false; `swimBack` returns false at stamina 0; water check: character below river surface with no seat → `getStamina` drain path uses water rate (setInWater covered by update test)
- [ ] **Step 2: Run specs, verify they fail**
- [ ] **Step 3: Implement** HealthService
- [ ] **Step 4: Run specs, verify PASS; Studio playtest**: damage a dummy → flinch; HP to 0 → downed + DownedEvent
- [ ] **Step 5: Commit** (ask approval first)
