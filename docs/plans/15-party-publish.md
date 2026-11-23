# Task 15: Party Setup + Publish

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** all prior tasks · **Produces:** playable published experience (Phase 1 complete)

**Goal:** Prove the game works for 1–4 players, tune what playtests expose, and publish it so you and your friends can play.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply (raft allocation rule verified live here).

**Files:**
- Modify: `src/server/Services/RaftService.luau` (allocation fixes found in testing), `src/server/Services/RunService.luau` (solo `selfReviveUsed` set at run start), config files for tuning
- Create: nothing new

**Interfaces:**
- Consumes: everything from Tasks 01–14.
- Produces: a published private Roblox experience meeting the milestone exit criteria below.

- [ ] **Step 1: Multi-client test** — Studio "Local Server" with 1, 2, 3, and 4 players: raft count and seats match the rule (solo 1 / duo 1×2 / trio 2+1 / quad 1 big); revive works in duo/trio/quad
- [ ] **Step 2: Full loop playtest** — solo run: second wind triggers once; duo run: revive returns teammate at 25 HP; both: checkpoints restore full HP; coins persist after leave/rejoin
- [ ] **Step 3: Fix tuning** found in playtests — config values only; note every changed value in design.md §18 change log
- [ ] **Step 4: Publish** — Studio → File → Publish to Roblox As; set to private; invite friends to test
- [ ] **Step 5: Commit** any changed files individually (ask approval first); push

## Milestone Exit Criteria (PRD M0+)

- 60-second playable loop that is fun to crash in (design.md pillar 1)
- HP/stamina/ragdoll/revive/checkpoints all work in a 2-player session
- Coins persist across rejoin; Frog purchasable and equippable
- Data survives 20 test sessions (PRD M1 criterion, tested here early)
