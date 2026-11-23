# Task 02: Shared Types, Math, Config

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** folder tree (Task 01) · **Produces:** `Math`, `GameConfig`, `EvolutionConfig`, `Types` → Tasks 04, 06, 07

**Goal:** The shared foundation every later task reads: global tuning numbers, evolution definitions, shared types, and pure math helpers.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply.

**Files:**
- Create: `src/shared/Types.luau`, `src/shared/Util/Math.luau`, `src/shared/Config/GameConfig.luau`, `src/shared/Config/EvolutionConfig.luau`
- Test: `src/tests/test_math.luau`, `src/tests/test_config.luau`

**Interfaces:**
- Consumes: nothing new.
- Produces:
  - `Math.clamp(value: number, min: number, max: number): number`
  - `Math.toDayKey(unixTime: number): string` — `"YYYY-MM-DD"` UTC
  - `Math.angleBetweenUp(cframe: CFrame): number` — tilt in degrees
  - `Math.formatDistance(studs: number): string`
  - `GameConfig` — every key from Global Constraints with its exact value
  - `EvolutionConfig.get(id: string): EvolutionDef?` — only `frog` for now: `{ id = "frog", name = "Frog", cost = 200, passive = { jumpMult = 1.2 }, abilityName = "DoubleJump", cooldown = 8 }`
  - `Types.PlayerProfile`, `Types.RunState` (fields include `hp`, `stamina`, `downed`, `checkpointIndex`, `selfReviveUsed`, `lives`, `coins`), `Types.RunResult`, `Types.EvolutionDef` — shaped per docs/functions.md §1.7 and docs/architecture.md §8

- [ ] **Step 1: Write failing tests**
  - `test_math.luau`: `Math.clamp(999, -1, 1) == 1`; `Math.clamp(-5, -1, 1) == -1`; `Math.toDayKey` returns the same string for two timestamps on the same UTC day; `Math.angleBetweenUp(CFrame.Angles(0, 0, 0)) == 0`
  - `test_config.luau`: every Global Constraints key exists on `GameConfig` with its exact value; `EvolutionConfig.get("frog").cost == 200`; `EvolutionConfig.get("nope") == nil`
- [ ] **Step 2: Run specs, verify they fail** (module not found)
- [ ] **Step 3: Implement** the four modules with the exact signatures above; `--!strict` in all
- [ ] **Step 4: Run specs, verify PASS**
- [ ] **Step 5: Commit** each file individually (ask approval first)
