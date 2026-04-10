# pve2 — Node 2

**Secondary node.** Hosts GPU-dependent services (Jellyfin, Immich ML) and cloud storage.

## Specs

| Component | Detail |
|---|---|
| **CPU** | Intel Core i7-4790 @ 3.60GHz (4c/8t) |
| **RAM** | 32GB |
| **GPU** | NVIDIA GeForce GTX 980 Ti (6GB VRAM) — passed through to jellyfin VM |
| **OS Disk** | 932GB SSD → Proxmox root (LVM) |
| **Flash Pool** | 932GB SSD → `flash` ZFS (same disk, partitioned) |
| **HDD** | 3.6TB HDD → additional storage |

## ZFS Pools

### `flash` — SSD pool
Single WD 1TB Blue SSD. Hosts container rootfs and VM disks.

```
flash   928G  used  ONLINE
└── ata-WDC_WD1002FAEX-00Y9A0 (single SSD)
```

## GPU Passthrough

The GTX 980 Ti is passed through entirely to the `jellyfin` VM (ID 219) using PCIe passthrough (`hostpci0: 0000:02:00.0`). Inside the VM:

- **Jellyfin** uses it for hardware video transcoding
- **Immich machine learning** uses it for CUDA-accelerated facial recognition and smart search (image embeddings via the `release-cuda` image)

NVIDIA driver version: 535.288.01 | CUDA: 12.2

## Containers & VMs

| ID | Name | Type | Role |
|---|---|---|---|
| CT 101 | nextcloud | LXC | Nextcloud AIO |
| CT 102 | claude | LXC | Claude Code MCP server |
| VM 219 | jellyfin | VM | Jellyfin · Immich · Jellyseerr · Jellystat |
| VM 300 | amp-gameserver | VM | AMP multi-game server (stopped) |
