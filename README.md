# Rapids Rush: Evolve or Drown

A Roblox obby-racer hybrid where players ride a raft down a chaotic river, collect coins, and spend them on creature evolutions that change how the raft handles the water.

## Documentation Index

| File | Purpose |
|------|---------|
| [PRD.md](PRD.md) | What we are building, for whom, and why. Features, priorities, success metrics. |
| [design.md](design.md) | Game design: core loop, controls, HP/stamina, revive, evolutions, obstacles, progression, balancing, UI. |
| [architecture.md](architecture.md) | Technical design: client/server split, services, data model, remote events, folder layout. |
| [functions.md](functions.md) | Function and module reference: every script and the functions it exposes. |
| [docs/DECISIONS.md](docs/DECISIONS.md) | Decision checkpoint: each major choice with status and sign-off. |
| [docs/plans/](docs/plans/) | Build plans: gapless, per-phase task breakdowns. |

## Quick Summary

- **Engine:** Roblox Studio
- **Language:** Luau
- **Genre:** Physics-based river racer with progression (inspired by *100 Miles Down the River*, *Hyper Realistic Ragdoll*, and *Ant Evolution*)
- **Players:** 1 to 4 per server (co-op is core: shared rafts by party size, teammate revive; solo gets one self-revive)
- **Session length:** 2 to 5 minutes per run
- **Survival layer:** HP bar, stamina bar, ragdoll on every damaging hit, checkpoints
- **Target platform:** Desktop, mobile, console (Roblox cross-platform)

## Development Phases

1. **Prototype:** One river section, raft movement, one obstacle, HP/stamina/ragdoll, coin pickup, checkpoints, score display.
2. **Alpha:** Full river course, three evolutions, revive/swim-back, data saving, basic UI, shop stub.
3. **Beta:** All evolutions, seasonal rivers, leaderboard, monetization, playtesting, balancing.
4. **Launch:** Polish, analytics review, live events, content updates.

## Folder Layout for This Repo

```
rapids-rush/
├── README.md
├── PRD.md
├── design.md
├── architecture.md
└── functions.md
```

The Roblox place itself is built in Studio. These files are the source of truth for design and code structure. Keep them updated when a decision changes.

## Rules for Updating the Docs

- Change the PRD first when a feature's scope changes.
- Update design.md when numbers (costs, speeds, durations) change.
- Update architecture.md and functions.md when a new script, remote event, or data field is added.
- Mark decisions with a date in the change log at the bottom of each file.
