# AMP Game Server

AMP (Application Management Panel) — multi-game server manager supporting dozens of game servers from a single interface.

## Setup

- **Host:** amp-gameserver VM (ID 300) on pve2
- **Status:** Currently stopped
- **Resources:** 4 cores, 16GB RAM

## Notes

AMP supports games like Minecraft, Valheim, ARK, Rust, and many others. The VM is kept stopped when not in use to free resources on pve2.

Game server data is stored on the media LXC's `tank_new` array at `/data/servers/AMP/`.
