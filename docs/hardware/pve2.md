# pve2 — Node 2

**Secondary node.** Hosts GPU-dependent services (Jellyfin, Immich ML) and cloud storage.

## Specs

| Component | Detail |
|---|---|
| **CPU** | Intel Core i7-4790 @ 3.60GHz (4c/8t) |
| **RAM** | 32GB |
| **GPU** | NVIDIA GeForce GTX 980 Ti (6GB VRAM) — shared via LXC device passthrough to CT 101, CT 103, CT 104 |
| **OS Disk** | 932GB SSD → Proxmox root (LVM) |
| **Flash Pool** | 932GB SSD → `flash` ZFS (same disk, partitioned) |
| **HDD** | 3.6TB HDD → additional storage |

## ZFS Pools

### `flash` — SSD pool
Single 1TB SSD. Hosts container rootfs and VM disks.

## GPU Passthrough

The GTX 980 Ti is shared across three LXC containers simultaneously via LXC device bind-mounts.

- **CT 103 (jellyfin)** — NVENC hardware transcoding (h264, hevc, av1)
- **CT 104 (immich)** — CUDA-accelerated ML inference (facial recognition, CLIP embeddings)
- **CT 101 (nextcloud)** — GPU available, not actively used yet

NVIDIA driver and CUDA toolkit installed on the host via NVIDIA's apt repo.

See [GPU Passthrough](../infrastructure/gpu-passthrough.md) for full config details.

## Containers & VMs

| ID | Name | Type | Role |
|---|---|---|---|
| CT 101 | nextcloud | LXC | Nextcloud AIO — GPU enabled |
| CT 102 | claude | LXC | Internal docs/wiki + Claude Code MCP server |
| CT 103 | jellyfin | LXC | Jellyfin · Jellyseerr · Jellystat — GPU enabled |
| CT 104 | immich | LXC | Immich (photos + CUDA ML) — GPU enabled |
| CT 105 | guacamole | LXC | Apache Guacamole (browser-based RDP/VNC) |
| VM 300 | amp-gameserver | VM | AMP multi-game server (stopped) |
| VM 301 | windows11-browser | VM | Windows 11 Pro for Windows-only browser tasks |
