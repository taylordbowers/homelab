# AdGuard Home

Network-wide DNS server with ad and tracker blocking.

## Setup

- **Host:** adguard LXC (CT 100) on pve-guide
- **DNS port:** 53
- **Web UI:** port 6060
- **Install:** Community Scripts (Proxmox VE Helper Scripts)

## Configuration

- **Upstream DNS:** Cloudflare (1.1.1.1) + Google (8.8.8.8)
- **Filtering:** Enabled with default blocklists
- **Plain DNS:** Enabled on port 53

All devices on the network point to AdGuard as their DNS server, configured via the router's DHCP settings.

## Installation (Proxmox)

Installed via the [Proxmox VE Community Scripts](https://community-scripts.github.io/ProxmoxVE/):

```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/adguard.sh)"
```

This creates a small Debian LXC (CT 100) with AdGuard Home pre-installed and configured to start on boot.
