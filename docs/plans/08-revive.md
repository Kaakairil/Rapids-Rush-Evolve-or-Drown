# Task 08: Revive + Swim-back Wiring

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** HealthService (07), party rafts (06) · **Produces:** complete revive loop → Tasks 09, 12 (downed/revive UI)

**Goal:** The co-op heart of the game: teammates revive downed players, swimmers climb back, solo players get a second wind.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply. **Review Focus line 3 lives here:** revive from an invalid state returns false with no side effects.

**Files:**
- Modify: `src/server/Services/HealthService.luau`, `src/shared/Net/Remotes.luau` (add `ReviveRequest` C→S RemoteEvent)
- Test: `src/tests/test_revive.luau`

**Interfaces:**
- Consumes: HealthService (Task 07), `RaftService.getRaft` (Task 06), profile `stats.revives`/`timesRevived` (Task 04).
- Produces:
  - `Remotes.get("ReviveRequest")` — C→S RemoteEvent, payload `downedPlayerUserId: number`
  - Server-side revive flow: client sends `ReviveRequest` → server validates (reviver in run, not downed; target downed; distance ≤ 12 studs) → enforces `REVIVE_HOLD_S` hold → restores target at `REVIVE_HP_PERCENT`, clears `downed`, fires `RevivedEvent(target, newHp)` → increments reviver's `stats.revives` and target's `stats.timesRevived`
  - `swimBack` wired to swim input: teleports to own raft seat, restores `REVIVE_HP_PERCENT` HP, consumes stamina cost
  - `selfRevive` auto-triggers inside `down()` when the player is solo (no teammates in run) and not yet used

- [ ] **Step 1: Write failing tests** — revive with target not downed returns false (no HP change); revive beyond 12 studs returns false; valid revive sets HP to 25, clears `downed`, fires `RevivedEvent`; revive increments `stats.revives` on reviver and `timesRevived` on target; solo `down()` auto-calls `selfRevive` (HP 25, `selfReviveUsed=true`)
- [ ] **Step 2: Run specs, verify they fail**
- [ ] **Step 3: Implement** proximity validation, hold timer, swim-back climb, solo auto second wind
- [ ] **Step 4: Run specs, verify PASS; Studio 2-player test**: knock p2 off, revive holds 3 s, HP returns at 25
- [ ] **Step 5: Commit** each file individually (ask approval first)
