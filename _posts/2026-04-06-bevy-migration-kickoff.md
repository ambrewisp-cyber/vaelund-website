---
title: "Devlog: Bevy Migration -- False Start, Real Progress"
date: 2026-04-06
tag: progress
excerpt: We tried to port the game client from macroquad to bevy. The compat layer broke. The clean implementation also broke. We deleted both clients instead.
---

**TL;DR** -- Started migrating the game client from macroquad to bevy. The compat layer approach didn't hold. Tried a clean bevy implementation. Then realized we didn't need either client right now, so we deleted both and kept only the server.

## What we did

- Began migrating the `client` crate from macroquad to bevy 0.13
- Added a `macroquad_compat.rs` layer to bridge existing draw calls to bevy's renderer
- Explored a separate `bevy_client` package as a clean implementation
- Deleted both `client/` and `bevy_client/` entirely -- 26,600 lines removed
- Kept `server/`, `auth/`, `editor/`, and `shared/` as the core packages

## Problems we hit

The compat layer was the wrong call. Macroquad's immediate-mode API and bevy's ECS model are fundamentally different -- forcing one onto the other created friction everywhere. Drawing a sprite meant fighting the bridge instead of using bevy properly.

The clean `bevy_client` implementation was the right instinct, but it surfaced a deeper question: the game is still at a stage where the client architecture is in flux. Investing heavily in a rendering pipeline when we haven't locked down the data model meant rewriting twice.

## What we learned

A compat layer buys you nothing when the underlying models don't align. Better to commit to a direction or step back entirely.

The game server is the anchor. Everything else is disposable until the core loop is solid.

## What's next

Focus stays on the server. The CLI client (UDP REPL) is enough to test gameplay. When the game loop is stable, we'll revisit the renderer with a clearer picture of what we actually need.
