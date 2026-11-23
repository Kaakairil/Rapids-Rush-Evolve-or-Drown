# Decision Checkpoint: Rapids Rush

**Purpose:** Confirm these decisions before starting the M0 prototype. Each item needs a status: **Approved**, **Changed**, or **Needs test**.

**Related:** [architecture.md](architecture.md), [design.md](design.md), [PRD.md](PRD.md), [functions.md](functions.md)

---

## Decision 1: River Current Is a Custom Force

**Decision:** Roblox does not simulate water flow. The river current is a constant custom force (`VectorForce`) applied to each raft. Terrain water is visual only and has no effect on physics.

**Why:** Roblox terrain water does not push parts. A custom force gives us predictable, tunable river speed, and it works the same on mobile and PC.

**Risks:**
- Rafts may look like they float on nothing if the visuals and force are not matched.
- Force scaling may feel different across sections with different speeds.
- Collisions with obstacles may feel too stiff or too loose.

**Checks before building:**
- [ ] Confirm the team accepts a fake current (visual flow is cosmetic, not physical).
- [ ] Build a 60-second test with one straight channel and verify the raft moves forward without input.
- [ ] Confirm that `VectorForce` and `AlignOrientation` keep the raft upright on flat water.

**Status:** [ ] Approved  [ ] Changed  [ ] Needs test

**Notes:**

---

## Decision 2: ProfileStore for Data Saving

**Decision:** Use ProfileStore (a session-locked wrapper around DataStoreService) instead of raw DataStoreService for player profiles.

**Why:** Raw DataStoreService can let two servers write to the same player's data at once, which causes data loss. ProfileStore locks each profile to one session and saves it safely on leave and on shutdown.

**Risks:**
- Adds an external module dependency that must be kept up to date.
- Requires careful handling when a profile fails to load (player gets a temporary profile).
- Learning curve for the team.

**Checks before building:**
- [ ] Install ProfileStore through Wally or the Toolbox, and confirm the version and license.
- [ ] Test a save/load round trip in a private server.
- [ ] Test rejoining quickly after leaving to confirm the session lock releases.
- [ ] Confirm that the `version` field and migration function work with a fake old profile.

**Status:** [ ] Approved  [ ] Changed  [ ] Needs test

**Notes:**

---

## Decision 3: Prices and Numbers Are Starting Values

**Decision:** All Robux prices, coin costs, forces, and thresholds in the docs are placeholders. They will be set in config files (`GameConfig`, `EvolutionConfig`, `ObstacleConfig`, `RiverConfig`) and tuned through playtests.

**Values that need testing:**

| Value | Starting Number | Where | Test Goal |
|-------|-----------------|-------|-----------|
| Current strength | 40 | `GameConfig.CURRENT_STRENGTH` | Run feels steady, not too fast or slow |
| Steer torque | 25 | `GameConfig.STEER_TORQUE` | Turning feels responsive but not twitchy |
| Knockoff impulse | 180 | `GameConfig.KNOCKOFF_IMPULSE` | Hard hits knock off, light hits do not |
| Flip threshold | 60 degrees | `GameConfig.FLIP_THRESHOLD_DEG` | Flips are rare but possible |
| Frog cost | 200 coins | `EvolutionConfig` | Bought within 3 to 4 runs |
| Turtle cost | 1,200 coins | `EvolutionConfig` | Feels like a meaningful goal |
| Average run coins | 60 to 120 | `RiverConfig` | Matches the coin economy targets |
| Gamepass prices | 199 / 349 / 249 Robux | Roblox dashboard | Players buy without feeling pressured |
| Coin pack prices | 25 / 99 / 299 Robux | Roblox dashboard | Value feels fair |
| Run length | 2 to 5 minutes | `RiverConfig` | Runs end at a satisfying point |

**Checks before building:**
- [ ] Put every number above in a config file, not hard-coded in scripts.
- [ ] Set up a playtest log (date, build, value changed, result).
- [ ] Plan at least 3 playtest sessions in the prototype phase and 5 in alpha.
- [ ] Decide who signs off on final prices before launch.

**Status:** [ ] Approved  [ ] Changed  [ ] Needs test

**Notes:**

---

## Decision 4: HP and Stamina Replace Instant Knockoff

**Decision:** Players have an HP bar (base 100) and a stamina bar (base 100). Obstacles deal light (~25) or heavy (~50) damage. Every damaging hit triggers a ragdoll flinch (heavier = longer). 0 HP downs the player in the water instead of instantly ending the run.

**Why:** The instant-knockoff model made the ragdoll a once-per-death moment. Per-hit ragdoll makes the comedy constant, and HP gives skilled players room to survive mistakes. Stamina creates a risk/reward rhythm: slow drain for on-raft dodges, fast drain for swimming and double jumps in water.

**Checks before building:**
- [ ] Confirm damage numbers feel right: light hits survivable, heavy hits scary.
- [ ] Confirm stamina drain rates: on-raft dodging is sustainable, swimming is a crisis.
- [ ] Confirm ragdoll flinch does not get annoying at ~3–6 hits per run.

**Status:** [ ] Approved  [ ] Changed  [ ] Needs test

**Notes:** Approved by owner 2026-11-23 (design direction confirmed in planning conversation; numbers pending playtest).

---

## Decision 5: Down, Revive, Swim-Back, Solo Second Wind

**Decision:** 0 HP downs the player (ragdolled in water, ~20 s bleed-out). Teammates can revive (3 s hold, returns at 20–25% HP). Downed players with stamina can swim back to their own raft. Solo players get one automatic self-revive per run at 25% HP. Full death respawns at the last checkpoint with full HP/stamina; losing all lives ends the run.

**Why:** Revive makes co-op the heart of the run, and the solo second wind mirrors it so solo never feels unfair. Swim-back keeps the downed state recoverable through player skill, not only through teammates. Checkpoints make death a setback, not a restart.

**Checks before building:**
- [ ] Test revive timing in a 2-player session: is 3 s hold right?
- [ ] Test solo second wind: does once per run feel fair?
- [ ] Test swim-back: can a player reliably reach their raft with partial stamina?

**Status:** [ ] Approved  [ ] Changed  [ ] Needs test

**Notes:** Approved by owner 2026-11-23 (revive, swim-back, and solo second wind confirmed in planning conversation).

---

## Decision 6: Party-Size Raft Allocation

**Decision:** One rule: one seat per player, one shared raft whenever the party fits within one raft. Solo → 1 small raft. Duo → 1 raft, 2 seats. Trio → 2 rafts (2 + 1). Quad → 1 big shared raft (4 seats).

**Why:** One rule covers all party sizes without per-size special cases in code. Shared rafts make revive physical and funny; the trio's 2+1 split naturally creates a "solo-ish" third player experience the owner explicitly asked for.

**Checks before building:**
- [ ] Build the 4-seat big raft and confirm it stays stable through rapids.
- [ ] Confirm the trio 2+1 split reads clearly to players (no confusion about who is where).

**Status:** [ ] Approved  [ ] Changed  [ ] Needs test

**Notes:** Approved by owner 2026-11-23 (solo behavior settled in planning conversation: small raft + second wind).

---

## Summary Table

| # | Decision | Status | Owner | Date Confirmed |
|---|----------|--------|-------|----------------|
| 1 | Custom river current force | | | |
| 2 | ProfileStore for data saving | | | |
| 3 | Numbers and prices are starting values | | | |
| 4 | HP and stamina replace instant knockoff | | | |
| 5 | Down, revive, swim-back, solo second wind | | | |
| 6 | Party-size raft allocation | | | |

## Sign-Off

Ready to start the M0 prototype when all decisions are marked **Approved** or **Changed** with notes.

**Signed off by:** ______________________ **Date:** ____________

## Change Log

| Date | Change |
|------|--------|
| 2026-11-23 | Created decision checkpoint file |
| 2026-11-23 | Added decisions 4–6 from planning conversation (HP/stamina, revive system, raft allocation) |
