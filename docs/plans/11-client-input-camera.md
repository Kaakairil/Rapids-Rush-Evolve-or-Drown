# Task 11: Client Input + Camera

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox syntax.
>
> **Consumes:** `Remotes` (03), server run lifecycle (10) · **Produces:** `InputController`, `CameraController` → Task 12

**Goal:** You can actually steer the raft — keyboard, touch, and gamepad — with a chase camera that shakes on crashes.

## Global Constraints

See [00-overview.md](00-overview.md) — all apply (client sends input only, 10 Hz).

**Files:**
- Create: `src/client/Main.client.luau`, `src/client/Controllers/InputController.luau`, `src/client/Controllers/CameraController.luau`

**Interfaces:**
- Consumes: `Remotes.get("RunStart")`, `Remotes.get("SteerInput")`, `Remotes.get("KnockoffEvent")`, `Remotes.get("DataLoaded")` (Task 03); server run lifecycle (Task 10) accepting input. `Main.client.luau` waits for `DataLoaded` before initializing controllers.
- Produces (docs/functions.md §5.2 / §5.3):
  - `InputController.init()` — UserInputService bindings; `getSteer(): number` (-1 to 1), `getPaddle(): boolean`, `getBrake(): boolean`; `sendLoop()` — sends `SteerInput(steer, paddle, brake)` at 10 Hz while in a run; `setScheme(scheme: string)`
  - Bindings: keyboard A/D or Left/Right steer, Space paddle, Shift brake; touch — drag left/right screen halves steer, on-screen buttons paddle/brake; gamepad — left stick X, A paddle, B brake
  - `CameraController.init()`, `follow(raftModel)` — camera subject = raft, behind-and-above offset; `shake(intensity, duration)` — decays over time; `setFlashReduction(enabled)` — disables shake per settings
  - `Main.client.luau` — waits for remotes, inits controllers

- [ ] **Step 1: Implement** `InputController` (UserInputService, 10 Hz send loop — no more, per network budget)
- [ ] **Step 2: Implement** `CameraController` (RenderStepped follow, shake decay)
- [ ] **Step 3: Wire** `Main.client.luau` to init both
- [ ] **Step 4: Studio playtest**: steer with keyboard; touch buttons appear on mobile emulator; gamepad stick works; camera follows raft; knockoff shakes camera
- [ ] **Step 5: Commit** each file individually (ask approval first)
