# Task 03: Remotes and Signal

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** folder tree (Task 01) · **Produces:** `Remotes`, `Signal` → Tasks 07–13

**Goal:** Every remote event/function defined once, by name, with a clear error on unknown names.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply (remote names defined only here).

**Files:**
- Create: `src/shared/Net/Remotes.luau`, `src/shared/Util/Signal.luau`
- Test: `src/tests/test_remotes.luau`

**Interfaces:**
- Consumes: nothing new.
- Produces:
  - `Remotes.init()` — creates `ReplicatedStorage/Remotes` folder if missing
  - `Remotes.get(name: string): RemoteEvent | RemoteFunction` — errors with a clear message on unknown names
  - Exact names and kinds (docs/architecture.md §7.1):
    - C→S RemoteEvents: `RunStart`, `SteerInput`, `UseAbility`, `ResetCheckpoint` (plus `ReviveRequest`, added in Task 08)
    - C→S RemoteFunctions: `BuyEvolution`, `EquipEvolution`
    - S→C RemoteEvents: `RunState`, `RunEnded`, `KnockoffEvent`, `DownedEvent`, `RevivedEvent`, `DamageEvent`, `DataLoaded` (profile summary; enables menu buttons)
  - Test includes `DataLoaded`: `Remotes.get("DataLoaded")` is a RemoteEvent
  - `Signal.new()`, `Signal:Connect(fn)`, `Signal:Fire(...)`, `Signal:Wait()`, `Signal:Destroy()`

- [ ] **Step 1: Write failing test** — `Remotes.get("RunStart")` is a RemoteEvent; `Remotes.get("DamageEvent")` is a RemoteEvent; `Remotes.get("BuyEvolution")` is a RemoteFunction; calling `Remotes.get("Bogus")` errors with a message naming the bad string
- [ ] **Step 2: Run spec, verify it fails**
- [ ] **Step 3: Implement** `Remotes.luau` (one table: name → kind; lazily create instances under the folder) and `Signal.luau` (~40 lines, no dependencies)
- [ ] **Step 4: Run spec, verify PASS**
- [ ] **Step 5: Commit** each file individually (ask approval first)
