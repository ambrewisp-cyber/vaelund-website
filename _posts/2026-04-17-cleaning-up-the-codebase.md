---
title: "#3: Cleaning up the codebase"
date: 2026-04-17
tag: progress
excerpt: Split the server code into focused modules and reduced draw calls from 350 to 5.
---

**TL;DR** -- Moved the server code around to make it easier to work with, and brought the renderer draw calls down from 350 to 5.

## What I did

### Split the server code

The server code was all in one file doing everything, ticking, broadcasting, zone management, command dispatch. I split it into focused modules so each one has one job. character_spawn, chat, cleanup, debug, message_router, movement, save, tests, tick_broadcast, tick_regen, zone_tracker.

### Reduced draw calls

The renderer was submitting a separate mesh for each grid line, each path segment, each outline. For a 150x150 zone that adds up fast: 302 meshes for the base grid alone, plus chunk boundaries, path highlights, hover and target outlines, rebuilt every frame regardless of whether anything changed.

I batched all lines in each system into a single mesh using Bevy's `PrimitiveTopology::LineList`. Hover and target outlines now use change detection so they only rebuild when the hovered or targeted entity actually changes.

Result: 350 draw calls down to 5.

## Problems I hit

The split was straightforward since I already knew where the natural seams were. The draw call issue took some digging, the symptom was high frame time but the cause was in how sprites were being submitted to the render pipeline.

## What's next

The clean structure makes the remaining items on the todo list easier to tackle. The foundation is better now, next week should be more active.

---

This week was light because I was away.
