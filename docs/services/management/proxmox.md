# Proxmox VE

The hypervisor platform, running on both nodes as one cluster.

## Cluster

- **Cluster name:** `Homelab`
- **Nodes:** 2, quorate
    - **pve** (10.0.0.3): primary, runs nearly every workload
    - **pve-guide** (10.0.0.1): storage node, retiring
- **Version:** Proxmox VE 8.4 on Debian 12 (Bookworm)
- **Web UI:** port 8006 on each node, also fronted by NPM with TLS

!!! note "Two-node quorum"
    A two-node cluster loses quorum if either node goes down, which makes `/etc/pve` read-only on the survivor. For planned maintenance on one node, `pvecm expected 1` on the other keeps it writable. The old `two_node` corosync flag was removed during the July 2026 migration.

## Guests

| Node | LXCs | VMs |
|---|---|---|
| pve | 100 adguard · 101 nextcloud · 102 claude · 103 jellyfin · 104 immich · 105 guacamole · 106 vaultwarden · 107 jarvis | 119 mediaServer · 300 amp-gameserver · 301 windows11-browser |
| pve-guide | 121 portainer · 200 media | — |

## AI-assisted management (MCP)

The [Claude Code agent](../automation/claude-agent.md) in CT 102 manages the cluster through a Proxmox **MCP server** (an open-source ProxmoxMCP fork). It connects to the Proxmox API with a dedicated, cluster-wide API token, not the root password. It can list and inspect guests, take snapshots and backups, start and stop guests, and run commands inside VMs through the guest agent, all from a Claude conversation. Destructive actions (stop, delete, rollback) always need explicit human confirmation.

## Key Practices

- **Same-named `flash` pool on each node**, so containers migrate with `pct migrate` without storage remapping
- **Unprivileged LXCs by default.** Privileged only where Docker plus GPU or AppArmor needs it (jellyfin, immich, guacamole, vaultwarden)
- **LXC device passthrough for the GPU** (RTX A3000 → CT 101, 103, 104 at once). See [GPU Passthrough](../../infrastructure/gpu-passthrough.md)
- **`onboot: 1`** on every production guest
- **QEMU guest agent** on every VM, for clean shutdowns, IP reporting, and exec access
- **Nightly vzdump** of every guest. See [Backups](../../infrastructure/backups.md)
- **Mail relay** (Postfix → Gmail SMTP) on both nodes, so job failures actually reach an inbox
