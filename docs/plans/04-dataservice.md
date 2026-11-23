# Task 04: DataService (ProfileStore)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** `Types.PlayerProfile` (Task 02) · **Produces:** `DataService.*`, server boot → Task 05

**Goal:** Player profiles that survive leave/rejoin without data loss.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply (store key `"RapidsRush_v1"`, schema version 1, autosave 60 s).

**Files:**
- Create: `src/server/Services/DataService.luau`, `src/server/Main.server.luau`
- Vendor: ProfileStore into `src/shared/Vendor/ProfileStore/` (from its GitHub repo, release tag pinned)
- Test: `src/tests/test_data_service.luau`

**Interfaces:**
- Consumes: `Types.PlayerProfile` (Task 02).
- Produces (docs/functions.md §4.1):
  - `DataService.init()` — creates the ProfileStore with key `"RapidsRush_v1"`
  - `DataService.start()` — connects `Players.PlayerAdded` / `PlayerRemoving`
  - `DataService.loadProfile(player: Player): PlayerProfile?`
  - `DataService.getProfile(player: Player): PlayerProfile?`
  - `DataService.saveProfile(player: Player)`, `DataService.releaseProfile(player: Player)`
  - `DataService.migrate(profile: any): PlayerProfile` — version upgrades
  - `DataService.createDefaultProfile(): PlayerProfile`
  - `DataService.saveAll()` — shutdown hook
  - Default profile: `version=1, coins=0, lifetimeCoins=0, ownedEvolutions={}, equippedEvolution=nil, bestDistance={}, bestCrash=nil, dailyChallenge={dayKey="",challengeId="",completed=false}, settings={musicVolume=0.5,sfxVolume=0.5,controlScheme="auto",reduceFlashing=false,autoSteer=false}, gamepasses={}, stats={runs=0,knockoffs=0,finishes=0,revives=0,timesRevived=0}`
  - `Main.server.luau` — calls init/start on services in boot order (docs/architecture.md §5), sets `game:BindToClose` → `saveAll`

- [ ] **Step 1: Write failing test** — `createDefaultProfile()` returns every key above with those values (assert each); `migrate` passes a version-1 profile through unchanged; `migrate` on a fake version-0 profile (missing `revives`/`timesRevived`) fills them with 0 and bumps `version` to 1
- [ ] **Step 2: Run spec, verify it fails**
- [ ] **Step 3: Implement** DataService (session locking; temporary profile with saving disabled on load failure; 60 s autosave loop; BindToClose save-all) and `Main.server.luau`
- [ ] **Step 4: Run spec, verify PASS; Studio test**: join a private server, rejoin, profile loads (no error on second session)
- [ ] **Step 5: Commit** each file individually (ask approval first)
