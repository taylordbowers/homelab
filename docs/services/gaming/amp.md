# AMP Game Server

AMP (Application Management Panel) is a multi-game server manager that runs dozens of different game servers from one web interface.

## Setup

- **Host:** amp-gameserver VM (ID 300) on `pve`
- **IP:** 10.0.0.17 (pinned lease)
- **OS:** Ubuntu 24.04
- **Resources:** 4 cores, 16GB RAM, 32GB disk on `flash` (NVMe)
- **Web UI:** port 8080
- **Status:** running, autostarts on boot
- **Remote access:** joined to the Tailscale tailnet

## Notes

AMP supports Valheim, ARK, Rust, Palworld, and many other games.

!!! note "Minecraft stays on Crafty"
    AMP can host Minecraft too, but after evaluating it the existing Minecraft servers stayed on [Crafty Controller](crafty.md). AMP is for everything else.

The VM is included in the nightly vzdump backup.
