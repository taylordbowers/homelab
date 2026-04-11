# Proxmox VE

Hypervisor platform running on both nodes in a cluster configuration.

## Cluster

- **Name:** two-node cluster
- **Node 1:** pve-guide (10.0.0.1) — primary
- **Node 2:** pve2 (10.0.0.2) — secondary
- **Web UI:** `proxmox.taylorsfunlab.com` (proxied via NPM)

## Version

Proxmox VE 8.x running on Debian 12 (Bookworm).

## MCP Integration

A Claude Code MCP server runs in CT 102 (claude LXC) on pve2. This enables AI-assisted cluster management — creating/deleting VMs, checking status, running commands inside VMs — directly from a Claude conversation.

The MCP server connects to the Proxmox API and is configured in `~/.claude/` on the Claude Code client machine.

## Key Practices

- LXCs run on `flash` ZFS pool for fast subvolume creation and snapshotting
- LXC device passthrough for GPU (GTX 980 Ti → CT 101, CT 103, CT 104 simultaneously) and iGPU passthrough (→ mediaServer VM)
- `onboot: 1` set on all production containers/VMs for automatic startup
- QEMU Guest Agent installed on all VMs for proper shutdown handling and exec access
