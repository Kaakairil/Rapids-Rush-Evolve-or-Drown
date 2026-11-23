# Task 01: Rojo Project Scaffold

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** nothing (first task) · **Produces:** buildable Rojo tree → Task 02

**Goal:** A `default.project.json` that maps the `src/` tree into Roblox Studio containers, and the empty folder skeleton.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply.

**Files:**
- Create: `default.project.json`
- Create: folder skeleton `src/server/Services/`, `src/client/Controllers/`, `src/shared/Config/`, `src/shared/Net/`, `src/shared/Util/`, `src/tests/`, `src/shared/Vendor/` (empty, for ProfileStore/TestEZ in Task 04)

**Interfaces:**
- Produces: Rojo container mapping per docs/architecture.md §4 — `src/server` → `ServerScriptService`, `src/client` → `StarterPlayer/StarterPlayerScripts`, `src/shared` → `ReplicatedStorage/Shared`, `src/tests` → `ServerStorage/TestSpecs`.

- [ ] **Step 1: Install Rojo** — via Aftman (`aftman add rojo-rbx/rojo@7.4.4`) or download from GitHub releases. Check: `rojo --version` prints 7.x
- [ ] **Step 2: Write `default.project.json`** with the four mappings above (`name = "RapidsRush"`, `servePort` default)
- [ ] **Step 3: Create the folder skeleton** — one placeholder `init.luau`-style module per folder named after the service/controller that will live there (placeholder content: `return {}`)
- [ ] **Step 4: Verify** — run `rojo build -o RapidsRush.rbxlx`; expected: succeeds with no errors
- [ ] **Step 5: Commit** `default.project.json` and the skeleton (each file individually, ask approval first)
