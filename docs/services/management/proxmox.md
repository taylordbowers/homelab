# Proxmox VE

The hypervisor platform. Since October 2026 it runs on a single node.

## Cluster

- **Cluster name:** `Homelab`
- **Nodes:** 1 live
    - **pve** (10.0.0.3): runs every workload and holds the RAIDZ1 array
    - **pve-guide** (10.0.0.1): **powered off** 2026-10-24, still a cluster member for a short rollback window before `pvecm delnode`
- **Version:** Proxmox VE 8.4 on Debian 12 (Bookworm)
- **Web UI:** port 8006, also fronted by NPM with TLS

!!! note "Quorum with one node powered off"
    A two-node cluster loses quorum if either node goes down, which makes `/etc/pve` read-only on the survivor. Proxmox's guest autostart also **waits for quorum**, so after a reboot nothing would start until someone ran `pvecm expected 1`, which is a runtime setting that resets on every boot. That's a nasty surprise after an unattended power-loss reboot. The fix for the rollback window: give `pve` **2 of 3 votes** (`quorum_votes: 2` in `corosync.conf`, with `config_version` bumped), so it's quorate by itself. When the retired node is removed with `pvecm delnode`, the vote goes back to 1. The old `two_node` corosync flag was removed during the July 2026 migration.

## Guests

| Node | LXCs | VMs |
|---|---|---|
| pve | 100 adguard · 101 nextcloud · 102 claude · 103 jellyfin · 104 immich · 105 guacamole · 106 vaultwarden · 107 jarvis · 121 portainer · 200 media (+1 private) | 119 mediaServer · 300 amp-gameserver · 301 windows11-browser |

## AI-assisted management (MCP)

The [Claude Code agent](../automation/claude-agent.md) in CT 102 manages the cluster through a Proxmox **MCP server** (an open-source ProxmoxMCP fork). It connects to the Proxmox API with a dedicated, cluster-wide API token, not the root password. It can list and inspect guests, take snapshots and backups, start and stop guests, and run commands inside VMs through the guest agent, all from a Claude conversation. Destructive actions (stop, delete, rollback) always need explicit human confirmation.

## Key Practices

- **Same-named `flash` pool on each node**, so containers moved between nodes without storage remapping. CT 121 and CT 200 moved by `zfs send -p` straight into the same `flash` dataset names, then their configs were moved across
- **Unprivileged LXCs by default.** Privileged only where Docker plus GPU or AppArmor needs it (jellyfin, immich, guacamole, vaultwarden)
- **LXC device passthrough for the GPU** (RTX A3000 → CT 101, 103, 104 at once). See [GPU Passthrough](../../infrastructure/gpu-passthrough.md)
- **`onboot: 1`** on every production guest
- **QEMU guest agent** on every VM, for clean shutdowns, IP reporting, and exec access
- **Nightly vzdump** of every guest. See [Backups](../../infrastructure/backups.md)
- **Mail relay** (Postfix → Gmail SMTP) on the host, so job failures actually reach an inbox
