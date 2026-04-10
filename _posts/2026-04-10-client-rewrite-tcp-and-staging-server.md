---
title: "#2: Client rewrite, TCP, and staging server"
date: 2026-04-10
tag: progress
excerpt: Deleted the old client, wrote a new one from scratch, moved UDP to TCP, and got a staging server up for playtesting.
---

**TL;DR** -- Deleted the old client, wrote a new one from scratch, moved UDP to TCP, and got a staging server up for playtesting.

## What we did

### Client rewrite

The old client was becoming too messy and because we didn't review it properly the architecture became difficult to reason about, especially given our beginner level at Rust and ECS. Every change made the next one harder, and we never had a clear enough mental model to fix the structure. The macroquad-to-bevy compatibility layer is a good example: it wasn't a deliberate decision, it just accumulated, and once it was there it made everything around it worse. So we deleted it all and started over.

The new client uses bevy_ecs for game logic and bevy_egui for all UI. Login, register, character select, menus, HUD, tooltips. Egui does all of it without fighting the renderer. It's a much simpler stack and we understand every line of it.

### Networking: UDP to TCP

We initially chose UDP because of an old memory of using it in a multiplayer game 12 years ago. We went in blind, without thinking much about the pros and cons. We used that single UDP connection for everything: player movement, spell casting, chat messages, character creation. One connection for all of it.

After turning the brain on, we realized this makes no sense. If you send a chat message, you don't want to "maybe get it to its destination." You want it to arrive. Period. So we considered having two connections: UDP for movement, TCP for everything else. In the end we kept only one TCP connection. It's simpler, and internet is much faster and more reliable today than it was 20 years ago.

### Database architecture

We split the database into two files. world_content_schema.sql is 213 lines of table definitions, no data. You rebuild with `sqlite3 world_data.db < shared/sql/world_content_schema.sql`. world_content_data.sql is 589 lines of INSERT statements: the seed data for zones, mobs, NPCs, items, quests, abilities, and models. The server starts with a fresh world_data.db, the schema creates the tables, and the data file populates them. Auth lives separately in auth.db, which handles account registration and login tokens.

### AI tooling: MiniMax to Opus

We switched from MiniMax 2.7 to Opus for AI-assisted coding. MiniMax costs $0.30 per million input tokens. Opus costs $15. That's 50x more expensive. But Opus produces the same quality in roughly half the time, so for iterative coding work the cost per useful output ends up similar.

### Hardware

Up until now we were developing on a Steam Deck. The system partition is read-only, and working around that takes effort we didn't want to spend anymore. We switched to an old PC with a 13 year old CPU that was gifted to us about 10 years ago. It runs Linux and compiles Rust, which is all we need.

## What's next

The new client still needs to catch up with the old one: spell casting, inventory, character stats, attacking. Next week will be slow because of a work event, but the goal is to get a staging server up and running so friends can download the client and connect from their own machines.
