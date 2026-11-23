# PRD: Rapids Rush: Evolve or Drown

**Version:** 0.2 (draft)
**Owner:** Kairill
**Status:** Pre-production

---

## 1. Vision

Rapids Rush is a fast, funny, physics-driven river game where every run is different. Players ride a raft through rapids, crash into obstacles in ragdoll chaos, and grow stronger by spending coins on creature evolutions. The game should be easy to start in 5 seconds and deep enough to keep players chasing better runs for weeks.

## 2. Problem Statement

Many Roblox physics games have strong moments of chaos but weak long-term progression. Players try them once, laugh, and leave. Ant Evolution-style progression keeps players coming back, but it rarely pairs with physical, skill-based gameplay. Rapids Rush combines both: skill and physics create the fun in each run, and evolutions give a reason to return.

## 3. Goals

| Goal | Measure | Target (first 30 days) |
|------|---------|------------------------|
| Fun first run | Players who finish or crash their first run and start a second | 60% |
| Retention | Day-1 retention | 35% |
| Session length | Average session time | 12 minutes |
| Progression | Players who unlock at least one evolution | 70% |
| Monetization | Players who buy at least one gamepass or product | 3% |

## 4. Non-Goals

- No combat or PvP in v1.
- No voice chat features or custom user-generated levels in v1.
- No open world. The game is a linear course with branches.
- No trading between players in v1.

## 5. Target Audience

- **Primary:** Ages 10–16 who play physics and progression games (ragdoll, obby, and evolution titles).
- **Secondary:** Ages 17–24 who watch and play for fun crashes and clips.
- **Platform:** Mobile-first controls, with keyboard and gamepad support.

## 6. User Personas

**Maya (11, mobile):** Plays during breaks. Wants a game she can understand in 10 seconds and crash in a funny way. Spends Robux on cosmetics.

**Dev (15, PC):** Likes competitive leaderboards and unlocking rare creatures. Wants to show off a strong evolution to friends.

**Ravi (19, casual):** Watches clips of big crashes. Wants fast restarts and challenge modes.

## 7. Feature List

Priority key: **P0** = required for launch, **P1** = important, **P2** = nice to have.

| ID | Feature | Priority | Notes |
|----|---------|----------|-------|
| F01 | Raft movement and steering | P0 | Touch, keyboard, and gamepad input |
| F02 | River current (constant forward push) | P0 | Custom force, not built-in Roblox water |
| F03 | Obstacles (logs, rocks, barrels, waterfalls) | P0 | At least 4 types in alpha |
| F04 | Ragdoll on every damaging hit | P0 | Light flinch ~1 s, heavy crash ~2 s; knockoff sends player flying |
| F22 | HP bar and damage system | P0 | Base 100 HP; obstacles deal light (~25) or heavy (~50) damage |
| F23 | Stamina bar | P0 | Base 100; slow drain on-raft, fast drain in water |
| F24 | Down, revive, and swim-back | P0 | Teammate revive to 20–25% HP; swim back with stamina; solo second wind |
| F25 | Checkpoints | P0 | 3–4 zones per river; save position, small coin reward, full restore on respawn |
| F26 | Party-size raft allocation | P0 | Solo 1 raft, duo 1×2 seats, trio 2 rafts, quad 1 big raft |
| F05 | Coin collection | P0 | Coins spawn along the course |
| F06 | Run score and finish screen | P0 | Distance, coins, time, rank |
| F07 | Evolution system (3 at launch) | P0 | Frog, Beaver, Turtle |
| F08 | Coin spend and evolution unlock | P0 | Coins persist between sessions |
| F09 | Data saving (coins, evolutions, best runs) | P0 | Uses ProfileStore (see architecture.md) |
| F10 | Main menu and HUD | P0 | Play, Evolve, Shop, Settings; HUD shows HP, stamina, checkpoints |
| F11 | Leaderboard (best distance, daily) | P1 | OrderedDataStore |
| F12 | Seasonal rivers (spring, frozen, lava) | P1 | Map variants with different obstacles |
| F13 | Race mode (up to 4 players) | P1 | Shared river, placement ranking |
| F14 | Daily challenges | P1 | Rotating goals, bonus coins |
| F15 | Raft cosmetics | P1 | Color, sail, and decal options |
| F16 | Gamepass: extra evolution slot | P1 | Monetization |
| F17 | Gamepass: double coins | P1 | Monetization |
| F18 | Developer products: coin packs | P1 | Monetization |
| F19 | Rare evolutions (event-only) | P2 | Limited-time creatures |
| F20 | Friends and social features | P2 | Invite to race |
| F21 | Analytics events | P1 | Runs, deaths, revives, evolutions bought |

## 8. User Stories

- As a new player, I can press Play and be on a raft within 5 seconds.
- As a player, I can steer the raft left and right and feel the current push me forward.
- As a player, I can see my HP and stamina bars react to hits and boosts, and watch a ragdoll flinch every time I take damage.
- As a player, I can get knocked off and swim back to my raft if I still have stamina.
- As a downed player, I can be revived by a teammate and return at 20–25% HP.
- As a solo player, I get one automatic self-revive per run so death never feels unfair.
- As a player, I can pass a checkpoint and know my progress is saved, with a small coin reward.
- As a player, I can collect coins and see my total update on the HUD.
- As a player, I can open the Evolve menu, see what each creature does, and buy one with coins.
- As a player, I can come back tomorrow and still have my coins and evolutions.
- As a racer, I can join a 4-player race and see my placement in real time.
- As a player, I can check the leaderboard to see how my best distance compares.
- As a developer, I can see how many runs, deaths, and purchases happen each day.

## 9. Functional Requirements

**FR-1 Run lifecycle:** A run starts when the player presses Play. It ends when the raft reaches the finish line, the player dies with no revive available, or the player quits. Each run produces a result record.

**FR-2 HP, stamina, and lives:** Each player has an HP bar (base 100), a stamina bar (base 100), and 3 lives per run. Obstacles deal damage; every damaging hit triggers a ragdoll flinch (heavier hits ragdoll harder). Stamina drains slowly for on-raft actions (paddle boost, dodge) and fast for in-water actions (swimming, double jump). At 0 HP the player is downed in the water.

**FR-3 Down, revive, and respawn:** A downed player can be revived by a teammate (returns at 20–25% HP), can swim back to their own raft if they have stamina, or (solo) uses one automatic self-revive per run at 25% HP. Death with no revive available, or losing all lives, ends the run or respawns at the last checkpoint. Checkpoints (3–4 per river) save run position and give a small coin reward; respawn restores full HP and stamina.

**FR-4 Coins:** Coins are added to the run total when collected and to the persistent balance at the end of the run.

**FR-5 Evolution:** An evolution is equipped by spending coins. Only one evolution is active at a time in v1. The player can switch in the Evolve menu between runs.

**FR-6 Persistence:** Coins, owned evolutions, equipped evolution, best distance, and settings are saved between sessions.

**FR-7 Leaderboard:** The top 100 best distances per day and all-time are stored and shown in the menu.

**FR-8 Race mode:** Up to 4 players start together. Placement is based on distance traveled. Results give bonus coins to all players, with more for first place.

**FR-9 Anti-exploit:** Coin rewards, distance, and evolution purchases are validated on the server. The client only sends input, never results.

## 10. Non-Functional Requirements

- **Performance:** 60 FPS on mid-range mobile devices with up to 4 players in a server.
- **Load time:** The main menu loads in under 8 seconds on a typical mobile connection.
- **Stability:** No data loss on player leave. Data is saved on leave and every 60 seconds.
- **Accessibility:** Text readable at 14px minimum on mobile. Colors are not the only signal for dangerous obstacles (use shape and icons too).
- **Localization:** English at launch. All UI text stored in a single table so it can be translated later.
- **Safety:** No content that depicts real-world violence or harm. Ragdoll crashes are cartoonish and not gory.

## 11. Monetization

| Item | Type | Price (Robux, initial) | Notes |
|------|------|------------------------|-------|
| Extra Evolution Slot | Gamepass | 199 | Allows 2 active evolutions |
| Double Coins | Gamepass | 349 | Doubles coin rewards |
| VIP Raft Pack | Gamepass | 249 | Exclusive raft skin and trail |
| Coin Pack S/M/L | Developer product | 25 / 99 / 299 | Direct coin purchase |
| Evolution Crate | Developer product | 49 | Random rare evolution (event-only at launch) |

Prices are placeholders and will be tested before launch. No pay-to-win mechanic in races: all race evolutions are earned, and paid items only add convenience or cosmetic value.

## 12. Analytics Events

| Event | When | Data |
|-------|------|------|
| `run_started` | Run begins | player id, river id, equipped evolution |
| `run_ended` | Run ends | distance, coins, time, deaths, reason |
| `evolution_bought` | Coins spent | evolution id, coin cost, balance after |
| `purchase_completed` | Robux purchase | product id, price |
| `race_finished` | Race ends | player count, placement |

## 13. Milestones

| Milestone | Scope | Exit criteria |
|-----------|-------|---------------|
| M0 Prototype | F01, F02, F03 (2 obstacles), F05 | Team can play a 60-second loop and find it fun |
| M1 Alpha | F04, F06, F07 (3 evolutions), F08, F09, F10 | Save/load works across 20 test sessions |
| M2 Beta | F11, F12, F14, F15, F16, F17, F18, F21 | Monetization tested, no critical bugs in 2 weeks of playtests |
| M3 Launch | F13, F19 (if ready), polish | Performance targets met, store page ready |

## 14. Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Physics feels unpredictable | High | Clamp max velocity, use simple collision shapes, playtest weekly |
| Mobile performance drops with many obstacles | High | Object pooling, limit active obstacles per section, streaming |
| Players quit after the first crash | High | Fast restart (under 3 seconds), show a "best crash" stat |
| Data loss from bugs | High | ProfileStore with session locking, backup logs |
| Pay-to-win perception | Medium | Keep all race advantages earnable, show paid items clearly |
| Scope creep | Medium | Stick to P0 for alpha, P1 only after M1 is passed |

## 15. Out of Scope for v1

Custom levels, trading, clans, voice features, and any combat system.

## 16. Open Questions

- ~~Should lives be shared in race mode or separate per player?~~ (Resolved v0.2: HP/stamina/lives are always per player; revive is the co-op mechanic, not shared lives.)
- Do seasonal rivers reset progress, or is progress kept across seasons?
- What is the target number of evolutions at full launch (currently 12 planned)?

## 17. Change Log

| Date | Version | Change |
|------|---------|--------|
| 2026-11-23 | 0.1 | Initial draft from game idea |
| 2026-11-23 | 0.2 | Added HP/stamina/lives system (F22–F26), ragdoll on every hit, down/revive/swim-back, solo second wind, checkpoints. FR-2 and FR-3 rewritten. Race lives question resolved. |
