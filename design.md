# Game Design: Rapids Rush: Evolve or Drown

**Version:** 0.2 (draft)
**Related:** [PRD.md](PRD.md), [architecture.md](architecture.md), [functions.md](functions.md)

---

## 1. Design Pillars

1. **Instant fun:** The first crash should be funny within 30 seconds of playing.
2. **Readable chaos:** Players must understand why they crashed. Physics can be wild, but cause and effect should be clear.
3. **Earned growth:** Every evolution changes how a run feels, not just the numbers.
4. **One more run:** Restarts are fast, and each run ends with a clear reason to try again.

## 2. Core Gameplay Loop

```
 ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 │  Drift   │ -> │ Collect  │ -> │  Finish  │ -> │  Evolve  │
 │ (steer,  │    │ (coins,  │    │ or crash │    │ (spend   │
 │  dodge)  │    │  boosts) │    │ (results)│    │  coins)  │
 └──────────┘    └──────────┘    └──────────┘    └────┬─────┘
       ^                                              │
       └──────────────────── Restart ─────────────────┘
```

**Loop duration target:** 2 to 5 minutes per run. Restart in under 3 seconds.

## 3. Controls

| Input | Mobile | PC | Gamepad |
|-------|--------|----|---------|
| Steer left/right | Drag on screen left/right half | A / D or Left / Right | Left stick X |
| Paddle (speed boost) | Tap and hold bottom button | Space | A button |
| Brake / drift | Tap and hold top button | Shift | B button |
| Use evolution ability | Tap ability icon | E | X button |
| Swim / climb (in water) | Auto-swim toward raft, hold paddle to climb | Space to swim faster | A button |
| Revive teammate | Hold interact prompt near downed player | Hold F | Hold X |
| Reset to checkpoint | Button in pause menu | R | Start button |
| Pause | Top-right icon | Esc | Start button |

**Steering feel:** Steering applies torque to the raft, not a direct position change. The current keeps pushing forward, so the raft drifts. Players must correct to stay on course.

## 4. The Raft

- **Base size:** About 6 × 1 × 4 studs (raft Part plus a Model of planks).
- **Mass:** Light enough that a log can push it sideways, heavy enough that it does not flip from small hits.
- **Stability:** Raft keeps upright with a stabilizing force. A flip takes a strong hit (over the threshold in design.md section 7).
- **Seats and raft allocation:** One rule covers all party sizes — raft = one seat per player, plus one shared raft whenever the party fits within one raft:
  - Solo → 1 small raft (1 seat)
  - Duo → 1 raft with 2 seats
  - Trio → 2 rafts (2 + 1)
  - Quad → 1 big shared raft (4 seats)

## 5. River Structure

A run is a sequence of **sections**. Each section lasts 15 to 30 seconds and has one dominant challenge.

| Section | Name | Challenge | Introduced |
|---------|------|-----------|------------|
| 1 | Calm Start | Learn steering, collect coins | Start |
| 2 | Logs | Dodge floating logs | Start |
| 3 | Rocks | Avoid rock clusters, narrow channel | Start |
| 4 | Rapids | Fast current, sharp turns | Start |
| 5 | Barrels | Rolling barrels that knock sideways | Alpha |
| 6 | Waterfall | Short drop, boost to clear gap | Alpha |
| 7 | Whirlpool | Pulls raft in a circle, paddle to escape | Beta |
| 8 | Finish Falls | Final big drop, finish line | Start |

Each run uses 6 random sections from the pool, with section 1 always first and section 8 always last.

## 6. Obstacles

| Obstacle | Behavior | Hit Effect | Visual Cue |
|----------|----------|------------|------------|
| Log | Floats, slow drift | Small push (sideways) | Brown, bark texture |
| Rock | Static, sits in river | Medium push, may flip | Gray, jagged |
| Barrel | Rolls across channel | Large push, may knock off | Red with stripes |
| Waterfall | Fast drop, raft speeds up | Speed burst, needs boost | Blue-white mist |
| Whirlpool | Pulls toward center | Rotation force | Dark swirl with ring |
| Debris Net | Catches raft briefly | Stuck for 1.5 seconds | Green netting |

**Rule:** Every obstacle must have a visual cue visible 2 seconds before contact. Players should never be surprised by an unavoidable hit.

## 7. Crash, HP, Stamina, and Ragdoll

### 7.1 HP

- Every player has an HP bar per run (base 100).
- Obstacles deal damage on contact: light hits (log, small rock) deal ~25, heavy hits (barrel, waterfall plunge, flip) deal ~50.
- **Every damaging hit triggers a ragdoll flinch** — heavy attacks ragdoll harder and longer. The ragdoll chaos is constant feedback throughout the run, not just the death moment.
- At 0 HP the player is knocked off and downed.

### 7.2 Down, Revive, Death

- **Downed:** 0 HP → player is in the water, ragdolled, with a downed timer (~20 s before bleeding out to death).
- **Teammate revive:** A teammate on a raft can pull the downed player back aboard. Revive takes ~3 seconds of contact, and the revived player returns at 20–25% HP.
- **Swim back:** If the player still has stamina, they can swim back to their own raft and climb on — no teammate needed.
- **Solo second wind:** Solo players get one self-revive per run. It triggers automatically when they would die, returning them at 25% HP with a ragdoll first. Groups get more total revives (one per teammate), so co-op is rewarded but solo is not punished.
- **Death:** Downed with no revive available, or bled out → run ends (or respawn at checkpoint, see 7.4).

### 7.3 Stamina

- One shared stamina bar (base 100) with context-sensitive drain:
  - **On the raft:** paddle boosts and dodges drain stamina slowly (little by little) — dodging often is sustainable.
  - **Off the raft (in water):** swimming/running and double-jumping drain stamina much faster — falling off is a resource crisis. Climb back on before you are drained and helpless.
- Stamina regenerates when drifting calmly.
- At 0 stamina in water the player cannot swim effectively and drifts with the current.

### 7.4 Checkpoints

- The river is divided into 3–4 checkpoint zones, marked by flags/buoys.
- Crossing a checkpoint saves it for the run and gives a small coin reward.
- Death or being swept too far downstream respawns the player at their last checkpoint **with full HP and stamina** (position is saved, resources are not — respawning restores you to full, otherwise there is no tension left).
- Checkpoint progress is per player. A revived teammate rejoins at their own checkpoint, not the group's.
- Losing all lives (or dying with no revive available) ends the run.

### 7.5 Knockoff and Ragdoll

- **Knockoff threshold:** Impact force above 180 (impulse units) or a flip beyond 60 degrees from upright causes a knockoff.
- **Ragdoll:** Character switches to a ragdoll. Duration scales with hit force (light flinch ~1 s, heavy crash ~2 s).
- **Lives:** 3 lives per run. Losing all lives ends the run.
- **Comic feedback:** Each crash shows a short label ("SPLASH!", "FLIP!", "OUCH!") and a camera shake. Crash type is recorded for the player's stats.
- **Best crash:** The player's most powerful crash (by force) is saved and shown in the stats screen. This encourages players to try for funny crashes.

## 8. Coins and Items

| Item | Value | Spawn | Notes |
|------|-------|-------|-------|
| Coin | 1 | Scattered along river lines | Common |
| Golden Coin | 5 | Hidden in risky shortcuts | Rare |
| Fish | 3 | Near rocks | Shows a splash when collected |
| Fruit | Speed boost (3 seconds) | Near rapids | Helps clear waterfall gaps |
| Heart | +1 life | Rare, on barrels path | Max 5 lives |

**Coin economy target:**
- Average run gives 60 to 120 coins at start.
- Evolution 1 (Frog) costs 200 coins.
- Evolution 2 (Beaver) costs 600 coins.
- Evolution 3 (Turtle) costs 1,200 coins.
- Player should be able to buy Frog within 3 to 4 runs.

## 9. Evolutions

Only one is equipped at a time in v1. Each evolution has a passive effect and an active ability (cooldown-based).

| ID | Name | Cost | Passive | Active Ability | Cooldown |
|----|------|------|---------|----------------|----------|
| E01 | Frog | 200 | Jump height +20% | Double jump over one obstacle (costs stamina) | 8 s |
| E02 | Beaver | 600 | Builds temporary dam | Places dam that slows current for 4 s | 12 s |
| E03 | Turtle | 1,200 | Shell shield: blocks one hit per section (no ragdoll flinch either) | Shell up, 100% block for 2 s | 15 s |
| E04 | Otter | 900 | Faster paddle speed (+25%) | Dash forward 10 studs | 6 s |
| E05 | Crab | 1,000 | Stronger grip on raft (harder to knock off) | Pinch: pushes nearby debris away | 10 s |
| E06 | Penguin | 1,500 | Slides on ice sections | Belly-slide boost | 9 s |
| E07 | Duck | 800 | Lighter body, floats better | Flap to slow fall off raft | 5 s |
| E08 | Croc | 2,000 | +1 max life | Chomp: removes one obstacle in front | 20 s |
| E09 | Seal | 1,800 | Better steering responsiveness | Slip through narrow gaps | 12 s |
| E10 | Axolotl | 2,500 | Regenerates one lost life per run (once) | Heal 1 life, 30 s cooldown | 30 s |
| E11 | Gold Koi (event) | 3,000 | +50% coin pickup range | Golden burst: coins worth 3x for 5 s | 25 s |
| E12 | Dragon Turtle (rare) | Crate-only | Shell shield + 1 extra life | Shell burst pushes everything away | 30 s |

**Design rule:** Every evolution must change a decision the player makes in at least one section. If an evolution only adds numbers, it is redesigned.

## 10. Progression

- **Coins:** Earned per run and used to buy evolutions.
- **Evolution tree:** Evolutions are unlocked in order of cost, but the player chooses which to buy.
- **Best distance:** Each river tracks a personal best. Beating it gives bonus coins.
- **Daily challenge:** One goal per day (for example, "Collect 50 coins in one run" or "Survive Whirlpool without knockoff"). Completing it gives 150 coins.
- **Season pass (P2, later):** Not included in v1.

## 11. Seasonal Rivers

Rivers rotate every 2 weeks. Each has a unique palette, obstacle weights, and one special section.

| River | Theme | Special Feature |
|-------|-------|-----------------|
| Spring Flood | Green water, fast current | Floating flowers slow the raft |
| Frozen River | Ice, slippery surface | Ice patches increase slide distance |
| Lava Canyon | Orange and black, heat | Lava splash that deals knockoff chance |
| Night Rapids | Dark, lanterns | Lights show obstacle positions only at close range |

## 12. Co-op and Race Mode

Co-op is part of the core loop (revive requires teammates), not a separate mode.

- **Party sizes and rafts:** Solo → 1 small raft. Duo → 1 raft, 2 seats. Trio → 2 rafts (2 + 1). Quad → 1 big shared raft (4 seats). One rule: one seat per player, one shared raft whenever the party fits within one.
- **Revive:** A downed player (0 HP, in the water) can be pulled aboard by a teammate after ~3 s of contact; they return at 20–25% HP. Solo players get one automatic self-revive per run instead.
- **Start:** Countdown from 3, all rafts start together.
- **Placement (race):** Based on distance traveled. Ties go to the player with more coins.
- **Collisions:** Players can bump rafts. Bumps push the other raft slightly, but they never cause a knockoff.
- **Rewards:** 1st place: 300 coins. 2nd: 180. 3rd: 100. 4th: 50.
- **Evolutions:** Equipped evolutions apply in races. No race-only evolutions.

## 13. User Interface

### 13.1 Screens

| Screen | Contents |
|--------|----------|
| Main Menu | Play, Race, Evolve, Leaderboard, Shop, Settings |
| Lobby (Race) | Player list, ready button, countdown |
| HUD (in run) | HP bar, stamina bar, lives (hearts), coin count, distance bar, checkpoint flag, equipped evolution icon with cooldown |
| Pause | Resume, Reset to Checkpoint, Settings, Quit |
| Finish Screen | Distance, coins, time, rank, best crash, Play Again button |
| Evolve Menu | Grid of evolutions with cost, passive, active, Buy and Equip buttons |
| Leaderboard | Daily and All-Time tabs, top 100, player's own rank |
| Shop | Coin packs, gamepasses, VIP Raft Pack |
| Settings | Sound volume, music volume, control scheme, reduce flashing |

### 13.2 UI Rules

- All buttons are at least 44 × 44 pixels on mobile.
- Dangerous obstacles use shape and icon, not color alone.
- Coin count and lives are always visible during a run.
- Buttons are large and easy to reach on the bottom of the screen for mobile.
- The Evolve menu shows locked evolutions as silhouettes with their cost, so the player knows what they are working toward.

### 13.3 Feedback

- **Splash effect** on every water entry.
- **Screen shake** on hard hits, intensity based on force.
- **Coin sound** with pitch rising for each consecutive coin.
- **Crash label** pops up for 1 second.
- **Haptics** on mobile for knockoffs (if the device supports it).

## 14. Art and Audio Direction

- **Art style:** Bright, bold cartoon look with simple textures. Low-poly obstacles to keep performance high.
- **Color:** Each river has its own palette. The raft and player stay in the same hue family so the player is always easy to find.
- **Music:** Upbeat looping tracks, one per river. Music speeds up during rapids sections.
- **Sound design:** Splashes, creaks, and comedy sounds on crashes (boing, slide whistle). Keep sounds short to avoid fatigue.

## 15. Balancing Targets

| Metric | Target |
|--------|--------|
| Average run length | 2 to 5 minutes |
| Average knockoffs per run | 3 to 6 |
| First run finish rate | 15% |
| Players buying Frog within 1 hour | 60% |
| Coin income per hour (average) | 1,200 |
| Daily challenge completion | 40% of daily players |

Tune costs and section difficulty after each playtest session. Log results in the change log.

## 16. Accessibility

- Steering and paddling can be done with one button each.
- Option to reduce flashing and screen shake.
- Text size scales with device settings.
- Colorblind-friendly palette option (uses patterns along with colors).
- Auto-steer option in Settings for players who want to focus on crashes instead of control.

## 17. Open Design Questions

- Should the raft flip be recoverable by paddling, or always require a respawn?
- Is the Whirlpool section too punishing for new players? (Test in Alpha.)
- Should evolutions have upgrade levels (Level 1 to 3)? Not in v1.

## 18. Change Log

| Date | Version | Change |
|------|---------|--------|
| 2026-11-23 | 0.1 | Initial draft: loop, controls, obstacles, evolutions, UI |
| 2026-11-23 | 0.2 | Added HP bar, stamina bar, ragdoll on every hit, swim-back, teammate revive, solo second wind, raft allocation rule, checkpoints. Race Mode section became Co-op and Race Mode. |
