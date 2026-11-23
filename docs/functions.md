# Functions and Modules: Rapids Rush

**Version:** 0.2 (draft)
**Related:** [architecture.md](architecture.md), [design.md](design.md), [PRD.md](PRD.md)

This file lists every script module, the functions it exposes, and what each function does. Signatures use Luau type syntax. Items marked **(P1)** or **(P2)** are not needed for the prototype or alpha.

---

## 1. Shared Config

### 1.1 `GameConfig.luau`

Global numbers. Change tuning here only.

| Key | Type | Default | Purpose |
|-----|------|---------|---------|
| `STARTING_LIVES` | number | 3 | Lives per run |
| `MAX_LIVES` | number | 5 | Cap including hearts |
| `MAX_HP` | number | 100 | Base HP per run |
| `MAX_STAMINA` | number | 100 | Base stamina per run |
| `LIGHT_DAMAGE` | number | 25 | Light obstacle hit (log, small rock) |
| `HEAVY_DAMAGE` | number | 50 | Heavy hit (barrel, waterfall, flip) |
| `RAGDOLL_LIGHT_S` | number | 1 | Flinch duration, light hit |
| `RAGDOLL_HEAVY_S` | number | 2 | Ragdoll duration, heavy hit |
| `REVIVE_HP_PERCENT` | number | 25 | HP after revive (20–25%) |
| `REVIVE_HOLD_S` | number | 3 | Hold time to revive a teammate |
| `BLEED_OUT_S` | number | 20 | Downed timer before death |
| `SELF_REVIVE_HP_PERCENT` | number | 25 | HP after solo second wind |
| `STAMINA_DRAIN_RAFT` | number | 8 | Per second while paddling/dodging on raft |
| `STAMINA_DRAIN_WATER` | number | 25 | Per second while swimming/double jumping |
| `STAMINA_REGEN` | number | 15 | Per second while drifting calmly |
| `CURRENT_STRENGTH` | number | 40 | Base river push force |
| `STEER_TORQUE` | number | 25 | Turn force from input |
| `DRAG_COEFFICIENT` | number | 0.6 | Slows raft over time |
| `MAX_SPEED` | number | 60 | Studs per second cap |
| `FLIP_THRESHOLD_DEG` | number | 60 | Tilt that causes knockoff |
| `KNOCKOFF_IMPULSE` | number | 180 | Impact that causes knockoff |
| `RESPAWN_DELAY` | number | 2 | Seconds ragdolled before respawn |
| `CHECKPOINT_REWARD` | number | 10 | Coins per checkpoint crossed |
| `CHECKPOINT_COUNT` | number | 4 | Checkpoint zones per river |
| `AUTOSAVE_INTERVAL` | number | 60 | Seconds between saves |
| `INPUT_RATE_LIMIT` | number | 20 | Max input events per second |
| `FEATURE_FLAGS` | table | `{}` | Turns seasonal rivers and events on or off |

### 1.2 `EvolutionConfig.luau`

```lua
export type EvolutionDef = {
    id: string,
    name: string,
    cost: number,
    passive: { [string]: number },
    abilityName: string,
    cooldown: number,
    isRare: boolean?,
}

return {
    frog = { id = "frog", name = "Frog", cost = 200, passive = { jumpMult = 1.2 }, abilityName = "DoubleJump", cooldown = 8 },
    -- beaver, turtle, otter, crab, penguin, duck, croc, seal, axolotl, goldKoi, dragonTurtle
}
```

Exported helper: `EvolutionConfig.get(id: string): EvolutionDef?`

### 1.3 `ObstacleConfig.luau`

Stores size, mass, impact multiplier, and pool size per obstacle type.

### 1.4 `RiverConfig.luau`

Stores section list, spawn rules, and seasonal river definitions.

Exported helpers:
- `RiverConfig.getSection(id: string): SectionDef?`
- `RiverConfig.pickSections(count: number, rng: Random): {SectionDef}` **(sections 1 and 8 are fixed first and last)**
- `RiverConfig.getActiveRiver(season: number): RiverDef`

### 1.5 `Text.luau`

All UI strings keyed by ID.

- `Text.get(key: string, ...: any): string` formats a string with arguments.

### 1.6 `Remotes.luau`

Creates and returns all remote objects. Only file that calls `Instance.new("RemoteEvent")` for game remotes.

- `Remotes.get(name: string): RemoteEvent | RemoteFunction`
- `Remotes.init(): ()` creates folder `ReplicatedStorage/Remotes` if missing.

### 1.7 `Types.luau`

Exports: `PlayerProfile`, `RunState`, `RunResult`, `SectionDef`, `EvolutionDef`. `RunState` includes `hp`, `stamina`, `downed`, `checkpointIndex`, and `selfReviveUsed` fields.

---

## 2. Shared Utilities

### 2.1 `Signal.luau`

- `Signal.new(): Signal`
- `Signal:Connect(fn: (...any) -> ()): Connection`
- `Signal:Fire(...: any): ()`
- `Signal:Wait(): ...any`
- `Signal:Destroy(): ()`

### 2.2 `Pool.luau`

- `Pool.new(template: Instance, size: number, parent: Instance): Pool`
- `Pool:Get(): Instance` returns an available instance and makes it visible.
- `Pool:Return(instance: Instance): ()` hides and resets the instance.
- `Pool:Grow(count: number): ()` adds instances if needed (prototype only).

### 2.3 `Math.luau`

- `Math.clamp(value: number, min: number, max: number): number`
- `Math.lerp(a: number, b: number, t: number): number`
- `Math.angleBetweenUp(cframe: CFrame): number` returns tilt in degrees.
- `Math.toDayKey(unixTime: number): string` returns `"YYYY-MM-DD"` in UTC.
- `Math.formatDistance(studs: number): string` returns meters or km text.

---

## 3. Server Entry

### 3.1 `Main.server.luau`

Runs once at server start.

- Calls `init()` then `start()` on each service in boot order (see architecture.md section 5).
- Connects `Players.PlayerAdded` and `Players.PlayerRemoving` to `DataService` and `RunService`.
- Calls `game:BindToClose` to save all profiles.

---

## 4. Server Services

### 4.1 `DataService.luau`

Wraps ProfileStore.

- `DataService.init(): ()` creates the ProfileStore instance with the key `"RapidsRush_v1"`.
- `DataService.start(): ()` connects Players events for profile load and release.
- `DataService.loadProfile(player: Player): PlayerProfile?` loads and locks a profile. Returns nil on failure.
- `DataService.getProfile(player: Player): PlayerProfile?` returns the loaded profile or nil.
- `DataService.saveProfile(player: Player): ()` forces an immediate save.
- `DataService.releaseProfile(player: Player): ()` unlocks and saves on leave.
- `DataService.migrate(profile: any): PlayerProfile` upgrades old schema versions.
- `DataService.createDefaultProfile(): PlayerProfile` returns a new profile with defaults.
- `DataService.saveAll(): ()` saves every active profile (used on shutdown).

### 4.2 `CoinService.luau`

- `CoinService.getBalance(player: Player): number`
- `CoinService.add(player: Player, amount: number, reason: string): ()` logs the reason for analytics.
- `CoinService.spend(player: Player, amount: number): boolean` returns false if balance is too low.
- `CoinService.canAfford(player: Player, amount: number): boolean`
- `CoinService.applyMultiplier(player: Player, base: number): number` applies Double Coins if owned.

### 4.3 `ProgressionService.luau`

- `ProgressionService.getBestDistance(player: Player, riverId: string): number`
- `ProgressionService.recordRun(player: Player, result: RunResult): boolean` returns true if a new best distance was set.
- `ProgressionService.recordCrash(player: Player, force: number, kind: string): ()` updates best crash if stronger.
- `ProgressionService.getDailyChallenge(player: Player): {challengeId: string, completed: boolean}`
- `ProgressionService.checkDailyChallenge(player: Player, runState: RunState): ()`
- `ProgressionService.getStats(player: Player): {runs: number, knockoffs: number, finishes: number}`

### 4.4 `EvolutionService.luau`

- `EvolutionService.owns(player: Player, evolutionId: string): boolean`
- `EvolutionService.buy(player: Player, evolutionId: string): (boolean, string)` returns success and a reason text key.
- `EvolutionService.equip(player: Player, evolutionId: string): (boolean, string)`
- `EvolutionService.getEquipped(player: Player): EvolutionDef?`
- `EvolutionService.getPassive(player: Player): {[string]: number}` returns passive stats for a run.
- `EvolutionService.useAbility(player: Player, runState: RunState): boolean` checks cooldown, runs the ability, and starts the cooldown.
- `EvolutionService.getCooldownRemaining(player: Player): number`

### 4.5 `Abilities.luau` (under EvolutionService)

A table of ability functions. Each takes `(player: Player, runState: RunState, raft: Raft)`.

| Function | Evolution | Effect |
|----------|-----------|--------|
| `DoubleJump` | Frog | Applies upward impulse once more while airborne |
| `BuildDam` | Beaver | Spawns dam part that applies current reduction for 4 s |
| `ShellUp` | Turtle | Sets `runState.shieldActive = true` for 2 s |
| `Dash` | Otter | Applies forward impulse of 10 studs |
| `Pinch` | Crab | Pushes obstacles within 8 studs away |
| `BellySlide` | Penguin | Increases forward force for 1.5 s on ice |
| `Flap` | Duck | Applies upward force, reduces fall speed |
| `Chomp` | Croc | Destroys nearest obstacle in front |
| `SlipGap` | Seal | Temporarily ignores collision with narrow obstacles |
| `Heal` | Axolotl | Adds 1 life, capped at MAX_LIVES |
| `GoldenBurst` | Gold Koi (P2) | Coins worth 3x for 5 s |
| `ShellBurst` | Dragon Turtle (P2) | Pushes all nearby obstacles away |

### 4.6 `HealthService.luau`

- `HealthService.init(): ()` connects damage sources and revive remotes.
- `HealthService.getHp(player: Player): number`
- `HealthService.getStamina(player: Player): number`
- `HealthService.damage(player: Player, amount: number, heavy: boolean): ()` applies damage, triggers ragdoll flinch (duration from `RAGDOLL_LIGHT_S`/`RAGDOLL_HEAVY_S`), fires `DamageEvent`. Calls `down(player)` at 0 HP.
- `HealthService.heal(player: Player, amount: number): ()`
- `HealthService.drainStamina(player: Player, amount: number): ()`
- `HealthService.regenStamina(player: Player, amount: number): ()`
- `HealthService.down(player: Player): ()` sets downed state, starts bleed-out timer (`BLEED_OUT_S`), fires `DownedEvent`.
- `HealthService.revive(reviver: Player, downedPlayer: Player): boolean` validates proximity, holds `REVIVE_HOLD_S`, restores `REVIVE_HP_PERCENT`.
- `HealthService.swimBack(player: Player): boolean` lets a downed player with stamina climb onto their own raft at `REVIVE_HP_PERCENT`.
- `HealthService.selfRevive(player: Player): boolean` solo second wind, once per run, at `SELF_REVIVE_HP_PERCENT`.
- `HealthService.die(player: Player, kind: string): ()` final death — calls `RunService.loseLife`.
- `HealthService.respawnAtCheckpoint(player: Player): ()` teleports to last checkpoint with full HP and stamina.
- `HealthService.resetForRun(player: Player): ()` resets HP, stamina, self-revive flag, checkpoint index at run start.
- `HealthService.update(dt: number): ()` per-Heartbeat: bleed-out timers, stamina regen/drain ticks.

### 4.7 `RaftService.luau`

- `RaftService.spawn(player: Player, spawnCFrame: CFrame): Raft` creates the raft model from template and assigns it to the player.
- `RaftService.destroy(player: Player): ()`
- `RaftService.getRaft(player: Player): Raft?`
- `RaftService.applyCurrent(raft: Raft, direction: Vector3, strength: number): ()` sets `VectorForce`.
- `RaftService.applySteer(raft: Raft, steer: number): ()` applies torque from input.
- `RaftService.applyDrag(raft: Raft): ()`
- `RaftService.checkKnockoff(raft: Raft, impulse: number): (boolean, string)` returns true and kind (`"flip"`, `"impact"`) when knockoff happens.
- `RaftService.getSpeed(raft: Raft): number`
- `RaftService.enforceSpeedLimit(raft: Raft): ()`
- `RaftService.resetToCheckpoint(raft: Raft, checkpointCFrame: CFrame): ()`
- `RaftService.update(dt: number): ()` runs once per Heartbeat for all active rafts.

### 4.8 `ObstacleService.luau`

- `ObstacleService.init(): ()` creates pools for each obstacle type in the active river.
- `ObstacleService.spawnSection(riverId: string, sectionIndex: number, startZ: number): {Instance}` spawns obstacles for one section.
- `ObstacleService.despawnSection(sectionIndex: number): ()` returns obstacles to pool.
- `ObstacleService.update(dt: number, raftPosition: Vector3): ()` moves moving obstacles and manages the active window.
- `ObstacleService.getActiveWindow(raftZ: number): {number}` returns section indexes to keep alive.
- `ObstacleService.onHit(obstacle: Instance, raft: Raft, impulse: number): ()` fires `KnockoffEvent` if needed.
- `ObstacleService.getKind(obstacle: Instance): string` returns `"log"`, `"rock"`, `"barrel"`, etc.

### 4.9 `RiverService.luau` (P1 for seasonal rivers, basic version in prototype)

- `RiverService.load(riverId: string): Model` builds or clones the river terrain and sections into `Workspace/Rivers`.
- `RiverService.unload(): ()`
- `RiverService.getSectionCFrame(index: number): CFrame`
- `RiverService.getFinishLineCFrame(): CFrame`
- `RiverService.getCheckpoints(): {CFrame}`
- `RiverService.getCurrentDirection(): Vector3`

### 4.10 `RunService.luau`

- `RunService.init(): ()` connects `RunStart`, `SteerInput`, `UseAbility`, `ResetCheckpoint`, `ReviveRequest`.
- `RunService.start(): ()`
- `RunService.startRun(player: Player): boolean` checks profile, allocates raft by party size (solo 1 / duo 1×2 / trio 2+1 / quad 1 big), calls `HealthService.resetForRun`, creates run state, and begins countdown.
- `RunService.endRun(player: Player, reason: string): RunResult` builds result, saves, fires `RunEnded`.
- `RunService.getRunState(player: Player): RunState?`
- `RunService.handleSteer(player: Player, steer: number, paddle: boolean, brake: boolean): ()` validates and stores input.
- `RunService.loseLife(player: Player, kind: string): ()` reduces lives, ragdolls, schedules respawn.
- `RunService.respawn(player: Player): ()`
- `RunService.crossCheckpoint(player: Player, index: number): ()` saves checkpoint, grants `CHECKPOINT_REWARD`, fires `RunState`.
- `RunService.addCoins(player: Player, amount: number): ()` updates run coins and fires `RunState`.
- `RunService.checkFinish(player: Player): ()` ends run on finish line.
- `RunService.update(dt: number): ()` updates timers and distance for all active runs.
- `RunService.isInRun(player: Player): boolean`

### 4.11 `RaceService.luau` (P1)

- `RaceService.joinQueue(player: Player): ()`
- `RaceService.leaveQueue(player: Player): ()`
- `RaceService.createLobby(players: {Player}): RaceLobby`
- `RaceService.startCountdown(lobby: RaceLobby): ()`
- `RaceService.updatePlacements(lobby: RaceLobby): ()` runs every 0.5 s.
- `RaceService.finishRace(lobby: RaceLobby): ()` pays rewards and logs analytics.
- `RaceService.onPlayerLeft(player: Player): ()` removes from lobby, marks result as DNF.

### 4.12 `LeaderboardService.luau` (P1)

- `LeaderboardService.submit(player: Player, riverId: string, distance: number): ()` writes to daily and all-time boards.
- `LeaderboardService.getTop(scope: string, count: number): {{name: string, score: number, rank: number}}`
- `LeaderboardService.getPlayerRank(player: Player, scope: string): number?`
- `LeaderboardService.refresh(): ()` clears cache every 60 s.

### 4.13 `MonetizationService.luau` (P1)

- `MonetizationService.init(): ()` sets `MarketplaceService.ProcessReceipt`.
- `MonetizationService.checkGamePasses(player: Player): ()` updates cached ownership.
- `MonetizationService.ownsGamePass(player: Player, passId: number): boolean`
- `MonetizationService.processReceipt(receiptInfo: {PlayerId: number, ProductId: number}): Enum.ProductPurchaseDecision`
- `MonetizationService.grantProduct(player: Player, productId: number): boolean`

### 4.14 `AnalyticsService.luau` (P1)

- `AnalyticsService.logEvent(name: string, data: {[string]: any}): ()` sends the event through AnalyticsService.
- `AnalyticsService.logRunStart(player: Player, riverId: string, evolutionId: string?): ()`
- `AnalyticsService.logRunEnd(player: Player, result: RunResult): ()`
- `AnalyticsService.logEvolutionBought(player: Player, evolutionId: string, cost: number, balanceAfter: number): ()`
- `AnalyticsService.logPurchase(player: Player, productId: number, price: number): ()`
- `AnalyticsService.logRevive(reviver: Player, downedPlayer: Player, kind: string): ()` kind is `"teammate"`, `"swimBack"`, or `"selfRevive"`

---

## 5. Client Controllers

### 5.1 `Main.client.luau`

- Waits for `Remotes` and `DataLoaded`.
- Calls `init()` on each controller.
- Connects `RunEnded` to `HUDController.showFinish`.

### 5.2 `InputController.luau`

- `InputController.init(): ()` connects touch, keyboard, and gamepad input.
- `InputController.getSteer(): number` returns current steer value in -1 to 1.
- `InputController.getPaddle(): boolean`
- `InputController.getBrake(): boolean`
- `InputController.sendLoop(): ()` sends `SteerInput` at 10 Hz when in a run.
- `InputController.onTouchStart(position: Vector2): ()`
- `InputController.onTouchMove(position: Vector2): ()`
- `InputController.onTouchEnd(): ()`
- `InputController.setScheme(scheme: string): ()` switches between touch, keyboard, and gamepad settings.

### 5.3 `CameraController.luau`

- `CameraController.init(): ()`
- `CameraController.follow(raftModel: Model): ()` sets camera subject and offset.
- `CameraController.shake(intensity: number, duration: number): ()`
- `CameraController.setFlashReduction(enabled: boolean): ()` disables shake when setting is on.

### 5.4 `HUDController.luau`

- `HUDController.init(): ()`
- `HUDController.updateRunState(state: RunState): ()` updates HP bar, stamina bar, hearts, coins, distance bar, checkpoint flag.
- `HUDController.showDowned(timeLeft: number): ()` bleed-out timer and revive prompt while downed.
- `HUDController.showReviveProgress(progress: number): ()` hold-to-revive ring.
- `HUDController.showCrashLabel(text: string): ()` shows "SPLASH!" etc. for 1 second.
- `HUDController.updateAbilityCooldown(remaining: number, total: number): ()`
- `HUDController.showFinish(result: RunResult): ()` opens finish screen.
- `HUDController.hideAll(): ()`

### 5.5 `MenuController.luau`

- `MenuController.init(): ()`
- `MenuController.open(screenName: string): ()`
- `MenuController.close(screenName: string): ()`
- `MenuController.refreshEvolveMenu(): ()` reads profile summary and updates grid.
- `MenuController.onBuyPressed(evolutionId: string): ()` calls `BuyEvolution` RemoteFunction.
- `MenuController.onEquipPressed(evolutionId: string): ()` calls `EquipEvolution` RemoteFunction.
- `MenuController.refreshLeaderboard(scope: string): ()` calls `RequestLeaderboard`.
- `MenuController.promptGamePass(passId: number): ()` calls `MarketplaceService:PromptGamePassPurchase`.
- `MenuController.promptProduct(productId: number): ()` calls `MarketplaceService:PromptProductPurchase`.

### 5.6 `EffectsController.luau`

- `EffectsController.init(): ()`
- `EffectsController.playSplash(position: Vector3): ()`
- `EffectsController.playCoinPickup(position: Vector3, streak: number): ()` raises pitch for combo.
- `EffectsController.playHitEffect(position: Vector3, force: number): ()`
- `EffectsController.playRagdollFlinch(duration: number): ()` matches server ragdoll duration.
- `EffectsController.playAbilityEffect(evolutionId: string, position: Vector3): ()`
- `EffectsController.setReduceFlashing(enabled: boolean): ()`

---

## 6. Startup Sequence (Summary)

```
Server:
  Main.server -> DataService.init/start
              -> Progression, Evolution, Coin services
              -> RiverService, ObstacleService, RaftService
              -> RunService.init/start
              -> Race, Leaderboard, Monetization, Analytics

Client:
  Main.client -> wait for DataLoaded
              -> InputController.init (begins sendLoop)
              -> CameraController.init
              -> HUDController.init
              -> MenuController.init
              -> EffectsController.init
```

---

## 7. Prototype Scope (Subset)

For the M0 prototype, only these functions must exist:

- `GameConfig` (basic values)
- `Math.clamp`, `Math.angleBetweenUp`
- `RaftService.spawn` (seat allocation), `applyCurrent`, `applySteer`, `applyDrag`, `checkKnockoff`, `update`
- `ObstacleService.spawnSection` (two obstacle types), `update`, `onHit`
- `RiverService.load` (single river), `getSectionCFrame`, `getCheckpoints`
- `HealthService.damage`, `down`, `revive`, `swimBack`, `selfRevive`, `resetForRun`, `update` (HP, stamina, ragdoll flinch, checkpoints)
- `RunService.startRun`, `handleSteer`, `loseLife`, `respawn`, `crossCheckpoint`, `addCoins`, `update`, `endRun` (local only, no saving)
- `InputController.init`, `getSteer`, `sendLoop`
- `HUDController.updateRunState` (HP + stamina bars), `showCrashLabel`, `showDowned`

## 8. Test Checklist (Functions)

- [ ] `CoinService.spend` returns false when balance is too low and does not change balance.
- [ ] `EvolutionService.buy` rejects an already-owned evolution.
- [ ] `EvolutionService.useAbility` is ignored during cooldown.
- [ ] `DataService.migrate` upgrades a version 0 profile to the current version without losing coins.
- [ ] `RaftService.checkKnockoff` returns true above `KNOCKOFF_IMPULSE`.
- [ ] `RunService.handleSteer` clamps values outside -1 to 1.
- [ ] `HealthService.damage` triggers ragdoll flinch and fires `DamageEvent`; 0 HP calls `down`.
- [ ] `HealthService.revive` returns the downed player at `REVIVE_HP_PERCENT` HP and clears the downed state.
- [ ] `HealthService.selfRevive` works once per run only; second call returns false.
- [ ] `HealthService.swimBack` fails when stamina is 0.
- [ ] `RunService.crossCheckpoint` saves index and grants `CHECKPOINT_REWARD` only for new checkpoints.
- [ ] `Remotes.get` errors clearly on an unknown name.
- [ ] `Math.toDayKey` returns the same key for two timestamps on the same UTC day.

## 9. Change Log

| Date | Version | Change |
|------|---------|--------|
| 2026-11-23 | 0.1 | Initial function reference |
| 2026-11-23 | 0.2 | Added HealthService, new GameConfig keys (HP, stamina, damage, revive, checkpoints), RunService.crossCheckpoint, revive analytics, HUD downed/revive UI, updated prototype scope and test checklist |
