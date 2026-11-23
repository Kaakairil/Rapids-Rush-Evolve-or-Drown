# Architecture: Rapids Rush: Evolve or Drown

**Version:** 0.2 (draft)
**Related:** [PRD.md](PRD.md), [design.md](design.md), [functions.md](functions.md)

---

## 1. Overview

Rapids Rush is a Roblox experience built in Luau. It uses a **server-authoritative** model: the server owns coins, evolutions, run results, and race placement. The client handles input, camera, UI, and visual effects. The client never sends rewards or positions as final results.

```
┌────────────────────────── CLIENT (per player) ──────────────────────────┐
│  InputController   CameraController   HUDController   EffectsController │
│        │                  │                 │                 │         │
│        └──────────────────┴────── RemoteEvents / RemoteFunctions ──┐     │
└───────────────────────────────────────────────────────────────────┼─────┘
                                                                    │
┌──────────────────────────── SERVER ───────────────────────────────┼─────┐
│  RunService (run lifecycle)   RaftService (physics, current)  <───┘     │
│  ObstacleService   CoinService   EvolutionService                       │
│  HealthService (HP, stamina, ragdoll, revive)                           │
│  RaceService   ProgressionService   DataService (ProfileStore)          │
│  LeaderboardService   MonetizationService   AnalyticsService            │
└─────────────────────────────────────────────────────────────────────────┘
                    │                          │
            ProfileStore (saved data)   OrderedDataStore (leaderboards)
```

## 2. Technology Choices

| Area | Choice | Reason |
|------|--------|--------|
| Language | Luau with `--!strict` mode | Type checking catches errors early |
| Physics | Roblox physics with custom forces (`VectorForce`, `AlignOrientation`) | Built-in, works on mobile |
| Data saving | ProfileStore (session-locked wrapper around DataStoreService) | Prevents data loss from duplicate sessions |
| Leaderboards | OrderedDataStoreService | Built for ranked data |
| Monetization | MarketplaceService (gamepasses, developer products) | Standard Roblox store |
| Networking | RemoteEvent for one-way events, RemoteFunction for request/response (sparingly) | Simple and standard |
| UI | ScreenGui with UIListLayout and UIGridLayout | Responsive on mobile |
| Animation | TweenService for UI, Animator for characters | Built-in, smooth |
| Analytics | AnalyticsService (custom events) | Built-in Roblox analytics |
| Source control | Git + Rojo (sync Studio with files) | Editing in VS Code, version control |
| Testing | TestEZ (unit tests for pure logic) | Tests coins, costs, data schema |

**Important note on water:** Roblox does not simulate water flow. The river current is a custom constant force applied to each raft. Terrain water is used for visuals only. This is the core technical decision for the game.

## 3. Repository and Tooling

Use **Rojo** to keep source files in a folder and sync them into Roblox Studio. This makes the project easier to version and review.

```
rapids-rush/
├── default.project.json        # Rojo project mapping
├── src/
│   ├── server/                 # -> ServerScriptService
│   │   ├── Main.server.luau
│   │   ├── Services/
│   │   │   ├── RunService.luau
│   │   │   ├── RaftService.luau
│   │   │   ├── ObstacleService.luau
│   │   │   ├── CoinService.luau
│   │   │   ├── EvolutionService.luau
│   │   │   ├── RaceService.luau
│   │   │   ├── ProgressionService.luau
│   │   │   ├── DataService.luau
│   │   │   ├── LeaderboardService.luau
│   │   │   ├── MonetizationService.luau
│   │   │   └── AnalyticsService.luau
│   ├── client/                 # -> StarterPlayer/StarterPlayerScripts
│   │   ├── Main.client.luau
│   │   ├── Controllers/
│   │   │   ├── InputController.luau
│   │   │   ├── CameraController.luau
│   │   │   ├── HUDController.luau
│   │   │   ├── MenuController.luau
│   │   │   └── EffectsController.luau
│   ├── shared/                 # -> ReplicatedStorage/Shared
│   │   ├── Config/
│   │   │   ├── GameConfig.luau      # Global numbers (lives, speeds)
│   │   │   ├── EvolutionConfig.luau # Evolution costs and effects
│   │   │   ├── ObstacleConfig.luau  # Obstacle sizes, forces
│   │   │   ├── RiverConfig.luau     # Sections, seasonal rivers
│   │   │   └── Text.luau            # All UI text (localization ready)
│   │   ├── Net/
│   │   │   └── Remotes.luau         # Remote event definitions
│   │   ├── Types.luau               # Shared types (RunState, SaveData)
│   │   └── Util/
│   │       ├── Signal.luau
│   │       ├── Pool.luau            # Object pooling
│   │       └── Math.luau
│   └── tests/                  # TestEZ specs (not shipped to players)
└── assets/                     # Mesh, sound, UI image references
```

## 4. Roblox Container Mapping

| Folder in Studio | Contents | Replicates to Client |
|------------------|----------|----------------------|
| `ServerScriptService` | Server scripts and services | No |
| `ServerStorage` | Templates for rivers, obstacles, rafts | No |
| `ReplicatedStorage/Shared` | Config, types, remotes, utilities | Yes |
| `StarterPlayer/StarterPlayerScripts` | Client controllers | Yes (runs per player) |
| `StarterGui` | ScreenGuis for menus and HUD | Yes |
| `Workspace/Rivers` | Active river instance (server created) | Yes |
| `Workspace/Rafts` | Player rafts (server created) | Yes |
| `Workspace/Obstacles` | Spawned obstacles (server created) | Yes |

## 5. Server Services

Each service is a module with an `init()` (wires up events) and `start()` (begins work) function. `Main.server.luau` calls them in dependency order.

**Boot order:**
1. `DataService` (load player data first)
2. `ProgressionService`, `EvolutionService`, `CoinService`
3. `RiverService` (or `ObstacleService`) and `RaftService`
4. `RunService` (run lifecycle, uses all above)
5. `RaceService`, `LeaderboardService`, `MonetizationService`, `AnalyticsService`

| Service | Responsibility | Depends On |
|---------|----------------|------------|
| DataService | Load/save player profiles through ProfileStore | none |
| ProgressionService | Best distance, daily challenge progress, stats | DataService |
| CoinService | Add/spend coins, validate spends | DataService |
| EvolutionService | Owned/equipped evolution, activate abilities | CoinService, DataService |
| RaftService | Spawn raft, apply current force, steering torque, flip checks, seat allocation by party size | ObstacleService |
| HealthService | HP, stamina, damage, ragdoll flinch, downed state, revive, swim-back, self-revive | RaftService, RunService |
| ObstacleService | Spawn and move obstacles per section, collision callbacks | RiverConfig |
| RunService | Start/end run, lives, checkpoints, results | RaftService, HealthService, CoinService, ProgressionService |
| RaceService | Race lobby, countdown, placement, race rewards | RunService |
| LeaderboardService | Write and read leaderboard entries | DataService |
| MonetizationService | Gamepass checks, developer product receipts | CoinService, DataService |
| AnalyticsService | Send analytics events | none |

## 6. Client Controllers

| Controller | Responsibility |
|------------|----------------|
| InputController | Reads touch, keyboard, gamepad; sends input to server at 10 Hz |
| CameraController | Follows raft from behind, shakes on hard hits |
| HUDController | Updates hearts, coins, distance bar, evolution cooldown |
| MenuController | Opens and closes menus, handles Evolve and Shop UI |
| EffectsController | Splash particles, crash labels, sound triggers (client-only visuals) |

## 7. Client-Server Communication

### 7.1 Remote Definitions

All remotes live in `ReplicatedStorage/Shared/Net/Remotes.luau`. Names are never typed as strings outside this file.

| Remote | Type | Direction | Payload | Purpose |
|--------|------|-----------|---------|---------|
| `RunStart` | RemoteEvent | C -> S | none | Request a new run |
| `SteerInput` | RemoteEvent | C -> S | `steer: number (-1 to 1)`, `paddle: boolean`, `brake: boolean` | Player input (sent at 10 Hz) |
| `UseAbility` | RemoteEvent | C -> S | none | Activate evolution ability |
| `ResetCheckpoint` | RemoteEvent | C -> S | none | Respawn at last checkpoint |
| `BuyEvolution` | RemoteFunction | C -> S | `evolutionId: string` | Returns success/failure and new balance |
| `EquipEvolution` | RemoteFunction | C -> S | `evolutionId: string` | Returns success/failure |
| `RequestLeaderboard` | RemoteFunction | C -> S | `scope: "daily" \| "allTime"` | Returns top 100 entries |
| `RunState` | RemoteEvent | S -> C | `hp, stamina, lives, coins, distance, checkpoint, timeLeft` | Sent on change |
| `RunEnded` | RemoteEvent | S -> C | result table | Finish screen data |
| `KnockoffEvent` | RemoteEvent | S -> C | `force, kind` | Crash label and effects |
| `DownedEvent` | RemoteEvent | S -> C | `player, bleedOutTime` | Downed UI, revive prompt for teammates |
| `RevivedEvent` | RemoteEvent | S -> C | `player, newHp` | Revive feedback |
| `DamageEvent` | RemoteEvent | S -> C | `amount, position, ragdollDuration` | HP bar, hit flinch effects |
| `ObstacleHit` | RemoteEvent | S -> C | `obstacleId, position` | Hit visuals |
| `RaceState` | RemoteEvent | S -> C | placements, countdown | Race lobby and HUD |
| `DataLoaded` | RemoteEvent | S -> C | profile summary | Enables menu buttons |
| `ReviveRequest` | RemoteEvent | C -> S | `downedPlayerUserId: number` | Teammate revive request; hold time validated server-side |

### 7.2 Rules

- Client input is validated on the server: steering is clamped to -1 to 1, paddle and brake are booleans, and input from a player not in a run is ignored.
- The server runs the physics simulation. The client may predict visuals locally, but it accepts server positions.
- Rate limit: the server drops `SteerInput` if a player sends more than 20 events per second.
- Every RemoteFunction call has a 5-second timeout on the client.

## 8. Data Model

### 8.1 Player Profile (ProfileStore)

```lua
export type PlayerProfile = {
    version: number,            -- schema version for migrations
    coins: number,
    lifetimeCoins: number,
    ownedEvolutions: { [string]: boolean },
    equippedEvolution: string?,
    bestDistance: { [string]: number },   -- keyed by river id
    bestCrash: {
        force: number,
        kind: string,
        timestamp: number,
    }?,
    dailyChallenge: {
        dayKey: string,          -- "YYYY-MM-DD" in UTC
        challengeId: string,
        completed: boolean,
    },
    settings: {
        musicVolume: number,
        sfxVolume: number,
        controlScheme: string,
        reduceFlashing: boolean,
        autoSteer: boolean,
    },
    gamepasses: { [string]: boolean },   -- cache, always re-verified on load
    stats: {
        runs: number,
        knockoffs: number,
        finishes: number,
        revives: number,        -- times this player revived a teammate
        timesRevived: number,
    },
}
```

**Migration rule:** Each schema change increments `version` and adds a migration function in `DataService`. Old profiles are upgraded on load. Never remove a field without a migration.

### 8.2 Run Result (not saved directly, sent to player and analytics)

```lua
export type RunResult = {
    riverId: string,
    distance: number,
    coinsEarned: number,
    timeSeconds: number,
    knockoffs: number,
    finished: boolean,
    endReason: string,       -- "finished" | "outOfLives" | "quit"
    bestCrashForRun: { force: number, kind: string }?,
}
```

### 8.3 Leaderboard Keys

- Daily: `OrderedDataStore "Distance_" .. dayKey`
- All-time: `OrderedDataStore "Distance_AllTime"`
- Score = best distance in centimeters (integer) so ordering stays precise.

## 9. Core Systems

### 9.1 Run Lifecycle

```
Idle ──RunStart──> Countdown ──> Running ──┬── finish line ──> Finished
                                          ├── lives = 0 ────> Finished
                                          └── quit ─────────> Finished
Finished ──> results sent ──> profile saved ──> Idle
```

State is stored on the server in a `RunState` table per player. The client only reads copies.

### 9.2 Raft Physics

Each raft has:
- A **hull Part** (the physical body) with `CustomPhysicalProperties` for light weight.
- An **AlignOrientation** constraint to keep the raft upright unless it is flipped by a hard hit.
- A **VectorForce** for the river current (constant forward force, scaled by section speed).
- A **steering torque** applied each Heartbeat, based on the last `SteerInput`.
- A **drag force** opposite to velocity to cap top speed.

```
Each Heartbeat (server):
    current  = riverDirection * CURRENT_STRENGTH * section.speedMultiplier
    steer    = clamp(lastInput.steer, -1, 1) * STEER_TORQUE
    drag     = -velocity * DRAG_COEFFICIENT
    apply VectorForce(current + drag)
    apply torque(steer)
    if tilt > FLIP_THRESHOLD or impact > KNOCKOFF_THRESHOLD then knockoff()
```

Constants live in `GameConfig.luau`. Tune them in one place.

### 9.3 Obstacles

- Obstacles are created from templates in `ServerStorage/Templates/Obstacles`.
- Each section defines a spawn list (type, position offset, spacing).
- Obstacles are pooled: `Util/Pool.luau` reuses parts instead of destroying and creating them. Each river keeps a pool of 40 to 60 instances per obstacle type.
- Obstacles move with a server-side `Heartbeat` loop (logs drift, barrels roll) and update their CFrame. Only obstacles in the active window (current section plus next section) are simulated.
- Collision uses `Touched` only for triggers (coin, fish, checkpoint). Physical hits use the physics engine and check impulse on the raft.

### 9.4 Evolutions

- Each evolution is a config entry in `EvolutionConfig.luau` with id, cost, passive stats, and ability name.
- Passive effects are read when the run starts. Changes apply on the next run.
- Abilities have a cooldown tracked on the server. `UseAbility` is ignored if cooldown is not finished.
- Ability logic is a function table: `Abilities[evolutionId](player, runState)`. Adding an evolution means adding one config entry and one ability function.

### 9.5 Coins

- Coins are counted in the run state during play.
- At run end, the run total is added to the profile in one transaction.
- Spending is server-only: `CoinService.spend(player, amount)` returns success and checks the balance before deducting.
- Golden Coins and boosts multiply the value on the server, never on the client.

### 9.6 Race Mode

- A race lobby holds 2 to 4 players. Countdown starts when all players are ready or after 30 seconds with at least 2 players.
- Each race uses its own river instance so player rafts do not interfere with solo runs.
- Placement is calculated every 0.5 seconds from distance traveled and sent via `RaceState`.
- Race result gives coin rewards and logs `race_finished`.

### 9.7 Monetization

- Gamepass ownership is checked with `MarketplaceService:UserOwnsGamePassAsync` when a profile loads. Results are cached in the profile, but refreshed each session.
- Developer products use `ProcessReceipt`. The handler grants coins, saves the profile, and returns `PurchaseGranted` only after a successful save. This prevents lost purchases.
- Prompts are triggered from the client with `MarketplaceService:PromptGamePassPurchase` or `PromptProductPurchase`.

### 9.8 Data Saving

- DataService uses ProfileStore. Each profile is loaded with session locking, so the same player cannot be active on two servers.
- Auto-save runs every 60 seconds and on `PlayerRemoving`.
- If loading fails, the player gets a temporary profile with saving disabled and a message to rejoin.
- `game:BindToClose` saves all active profiles when the server shuts down.

### 9.9 Health, Stamina, and Revive

All state is server-side per player in the run state table; the client reads copies via `RunState`/`DamageEvent`.

- **HP:** Base 100, stored in run state. `HealthService.damage(player, amount, ragdollDuration)` applies damage, triggers the ragdoll flinch, and fires `DamageEvent`. At 0 HP the player is downed.
- **Ragdoll flinch:** Every damaging hit ragdolls the character. Duration scales with force (light ~1 s, heavy ~2 s). The ragdoll is applied server-side via `Humanoid` state changes / BallSocket constraints; client shows the effects.
- **Stamina:** Base 100. On-raft actions (paddle, dodge) drain slowly (e.g. 8/s while boosting); in-water actions (swim, double jump) drain fast (e.g. 25/s). Regenerates when drifting. At 0 stamina in water the player cannot swim effectively and drifts with the current.
- **Downed:** 0 HP → ragdolled in the water with a bleed-out timer (~20 s). `DownedEvent` shows the revive prompt to teammates.
- **Revive:** A teammate within reach holds the revive input for ~3 s → downed player is pulled aboard at 20–25% HP. Solo players get one automatic self-revive per run at 25% HP.
- **Swim-back:** A downed player with stamina > 0 can swim to their own raft and climb on at 20–25% HP.
- **Checkpoints:** 3–4 trigger zones per river. Crossing saves the player's checkpoint index and grants a small coin reward. Death or sweep-away respawns at the last checkpoint with full HP and stamina. Checkpoint progress is per player.

**Seat allocation:** `RaftService.spawn` reads the party size and builds the raft accordingly — 1 seat (solo), 2 seats (duo), 2+1 rafts (trio), 4-seat big raft (quad). One rule, no per-size special-casing beyond seat count.

## 10. Performance Plan

| Concern | Approach |
|---------|----------|
| Many obstacles | Object pooling, active window of 2 sections, despawn behind raft |
| Physics cost | Simple collision shapes (boxes and cylinders), no mesh colliders on moving parts |
| Mobile draw calls | Combine static river meshes, use low-poly models, limit particle emitters |
| Network traffic | Input at 10 Hz, state updates only on change, compressed numbers |
| Memory | Unload old river sections, clear run state on finish |
| Server load | Only simulate active rafts, one physics loop per server (not per raft) |

**Budgets:**
- Server Heartbeat under 8 ms with 4 players.
- Client frame time under 16 ms on a mid-range phone.
- Maximum 300 active parts per river section.

## 11. Security and Anti-Exploit

- All rewards are server-calculated. The client sends intent, not results.
- Input is rate limited and clamped.
- Coins and evolutions are changed only inside services.
- Remote names are defined once and checked for valid payload types.
- Speed hacks: server checks maximum speed each Heartbeat and resets the raft if it exceeds the limit.
- Teleport checks: if a raft moves more than the maximum distance in one frame, it is reset to the last valid position.
- Purchases are verified with `ProcessReceipt`, never with client-side claims.

## 12. Testing Strategy

| Test Type | What | Tool |
|-----------|------|------|
| Unit | Coin math, evolution costs, cooldowns, migrations | TestEZ |
| Integration | Save/load round trip, purchase flow | Studio test server |
| Physics | Raft behavior across sections (manual test checklist) | Studio playtest |
| Multiplayer | Race start, placement, disconnect during race | 2 to 4 Studio clients |
| Performance | Frame time and memory on mobile emulator | Studio MicroProfiler |
| Playtest | Fun and difficulty | Friends and test group |

## 13. Deployment

1. Develop in Studio with Rojo sync.
2. Test in private server before publishing.
3. Publish to a staging experience for playtests.
4. Publish to production when M2 exit criteria are met.
5. Feature flags (a config table in `GameConfig`) control new seasonal rivers and events, so they can be turned on or off without a code change.

## 14. Open Technical Questions

- Should race rivers be instanced per race or pooled and reused?
- Is one server per 4-player race needed, or can races share a server with solo runs?
- What is the maximum active raft count per server before physics cost is too high? (Test in Alpha.)

## 15. Change Log

| Date | Version | Change |
|------|---------|--------|
| 2026-11-23 | 0.1 | Initial architecture draft |
| 2026-11-23 | 0.2 | Added HealthService (HP, stamina, ragdoll flinch, down/revive/swim-back, checkpoints), new remotes (DownedEvent, RevivedEvent, DamageEvent), revive stats in profile, seat allocation in RaftService |
