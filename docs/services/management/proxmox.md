# Proxmox VE

Hypervisor platform running on both nodes in a cluster configuration.

## Cluster

- **Name:** two-node cluster
- **Node 1:** pve-guide (192.168.1.118) — primary
- **Node 2:** pve2 (192.168.1.157) — secondary
- **Web UI:** `proxmox.taylorsfunlab.com` (proxied via NPM)

## Version

Proxmox VE 8.x running on Debian 12 (Bookworm).

## MCP Integration

A Claude Code MCP server runs in CT 102 (claude LXC) on pve2. This enables AI-assisted cluster management — creating/deleting VMs, checking status, running commands inside VMs — directly from a Claude conversation.

The MCP server connects to the Proxmox API and is configured in `~/.claude/` on the Claude Code client machine.

## Key Practices

- LXCs run on `flash` ZFS pool for fast subvolume creation and snapshotting
- PCIe passthrough used for GPU (GTX 980 Ti → jellyfin VM) and iGPU (→ mediaServer VM)
- `onboot: 1` set on all production containers/VMs for automatic startup
- QEMU Guest Agent installed on all VMs for proper shutdown handling and exec access
