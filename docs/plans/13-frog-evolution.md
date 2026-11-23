# Task 13: Frog Evolution + Coin Pickups

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** `CoinService` (05), `EvolutionConfig` (02), remotes (03) · **Produces:** `EvolutionService.*`, `CoinPickupService` → Task 14 (buy/equip round-trip)

**Goal:** Coins on the river you can grab, and the Frog evolution you can buy — the earn-and-spend loop complete.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply (`EvolutionConfig.get("frog").cost == 200`).

**Files:**
- Create: `src/server/Services/EvolutionService.luau`, `src/server/Services/CoinPickupService.luau`
- Modify: `src/server/Main.server.luau` (boot order)
- Test: `src/tests/test_evolution.luau`

**Interfaces:**
- Consumes: `CoinService.*` (Task 05), `EvolutionConfig` (Task 02), `BuyEvolution`/`EquipEvolution`/`UseAbility` remotes (Task 03), `RaftService` (06) for ability impulses.
- Produces (docs/functions.md §4.4 / §4.5):
  - `EvolutionService.owns(player, id): boolean`; `buy(player, id): (boolean, string)` — rejects duplicate ownership (reason `"owned"`) and low balance (`"tooPoor"`); `equip(player, id): (boolean, string)` — fails for unowned; `getEquipped(player): EvolutionDef?`; `getPassive(player): {string: number}`; `useAbility(player, runState): boolean` — ignored during cooldown; `getCooldownRemaining(player): number`
  - `Abilities.DoubleJump(player, runState, raft)` — upward impulse once more while airborne; in water it drains stamina at the `STAMINA_DRAIN_WATER` rate (Review Focus line 1 context)
  - `CoinPickupService.init()` — spawns coin parts in lines along river sections; server-side pickup (`Touched`) → `RunService.addCoins(player, 1)`; coin respawns behind the raft window
- `buy` and `equip` are RemoteFunctions returning `(success, reasonKey)`.

- [ ] **Step 1: Write failing tests** — `buy("frog")` with 200 coins succeeds and balance drops to 0; second `buy("frog")` fails `"owned"`; `buy` with 199 coins fails `"tooPoor"`; `useAbility` twice within 8 s: second call returns false; `equip` for an unowned evolution fails; coin pickup adds 1 run coin
- [ ] **Step 2: Run specs, verify they fail**
- [ ] **Step 3: Implement** EvolutionService + `Abilities` table + CoinPickupService
- [ ] **Step 4: Run specs, verify PASS; Studio playtest**: collect 200+ coins in a run, buy Frog, double-jump over a log, cooldown enforced
- [ ] **Step 5: Commit** each file individually (ask approval first)
