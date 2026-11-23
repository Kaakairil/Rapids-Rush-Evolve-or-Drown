# Task 12: HUD + Effects

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** all S→C remotes (03), server events (07–10) · **Produces:** `HUDController`, `EffectsController` → Task 14 (finish screen extends HUD)

**Goal:** You can see your HP, stamina, coins, checkpoint progress, and the downed/revive state — the survival layer made visible.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply (buttons ≥ 44×44 px, shape+icon not color alone, per design.md §13.2).

**Files:**
- Create: `src/client/Controllers/HUDController.luau`, `src/client/Controllers/EffectsController.luau`

**Interfaces:**
- Consumes: `RunState`, `DamageEvent`, `DownedEvent`, `RevivedEvent`, `KnockoffEvent` (Task 03); server payloads (Tasks 07–10).
- Produces (docs/functions.md §5.4 / §5.6):
  - `HUDController.init()` — builds ScreenGui in code
  - `updateRunState(state)` — **HP bar**, **stamina bar**, hearts (lives), coin count, distance bar, checkpoint flag marker
  - `showDowned(timeLeft)` — bleed-out countdown + revive prompt; `showReviveProgress(progress)` — hold ring while reviving
  - `showCrashLabel(text)` — "SPLASH!" etc., 1 s; `updateAbilityCooldown(remaining, total)`; `showFinish(result)`; `hideAll()`
  - `EffectsController.init()`, `playSplash(position)`, `playCoinPickup(position, streak)` (pitch rises per combo), `playHitEffect(position, force)`, `playRagdollFlinch(duration)`, `playAbilityEffect(id, position)`, `setReduceFlashing(enabled)`
- UI rules (design.md §13.2): bars always visible during a run; text ≥ 14 px; danger shown with shape/icon too.

- [ ] **Step 1: Implement** HUDController (code-built ScreenGui; bars fill from `RunState` fields; downed overlay with countdown; revive hold ring)
- [ ] **Step 2: Implement** EffectsController (splash particles, coin sound with pitch streak, hit/ragdoll effects, crash labels)
- [ ] **Step 3: Studio playtest**: bars react to damage/boost; downed timer counts; teammate sees revive prompt; crash label pops on knockoff
- [ ] **Step 4: Commit** each file individually (ask approval first)
