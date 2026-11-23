# Task 05: CoinService

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** `DataService` (Task 04) · **Produces:** `CoinService.*` → Tasks 09, 13, 14

**Goal:** A coin balance that can only be changed through validated server calls.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply (spend validates amount > 0 and balance first — Review Focus line 5).

**Files:**
- Create: `src/server/Services/CoinService.luau`
- Test: `src/tests/test_coin_service.luau`

**Interfaces:**
- Consumes: `DataService.getProfile(player)` (Task 04) — profile field `coins`.
- Produces (docs/functions.md §4.2):
  - `CoinService.getBalance(player: Player): number`
  - `CoinService.add(player: Player, amount: number, reason: string)` — ignores `amount <= 0`
  - `CoinService.spend(player: Player, amount: number): boolean` — false and unchanged when `amount <= 0` or insufficient balance
  - `CoinService.canAfford(player: Player, amount: number): boolean`
  - `CoinService.applyMultiplier(player: Player, base: number): number` — applies Double Coins gamepass when owned (gamepass check itself is P1; without it returns base)

- [ ] **Step 1: Write failing tests** — profile with 50 coins: `spend(100)` returns false, balance still 50; `spend(30)` returns true, balance 20; `add(-5)` leaves balance unchanged; `spend(0)` returns false; `applyMultiplier` without gamepass returns base
- [ ] **Step 2: Run specs, verify they fail**
- [ ] **Step 3: Implement** CoinService
- [ ] **Step 4: Run specs, verify PASS**
- [ ] **Step 5: Commit** (ask approval first)
