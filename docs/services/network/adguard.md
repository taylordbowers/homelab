# AdGuard Home

Network-wide DNS server with ad and tracker blocking.

## Setup

- **Host:** adguard LXC (CT 100) on `pve`
- **IP:** 10.0.0.5 (static)
- **DNS port:** 53
- **Web UI:** port 80
- **Resources:** 1 core, 512MB RAM, 4GB disk (unprivileged Debian 12 LXC)
- **Install:** Community Scripts (Proxmox VE Helper Scripts)

## Configuration

- **Upstream DNS:** Cloudflare (1.1.1.1)
- **Filtering:** Enabled with default blocklists
- **Plain DNS:** Enabled on port 53
- **DNS rewrites:** internal hostnames (e.g. LAN-only dashboards) resolve to Nginx Proxy Manager, so they never need a public DNS record

## Who uses it

Every server guest in the cluster uses AdGuard as its **primary** resolver, with the ISP gateway as a **fallback**. Internal names resolve, and an AdGuard outage doesn't break the lab.

The ISP gateway won't let DHCP hand out a custom DNS server, so ordinary household clients currently use the gateway's resolver and skip AdGuard's blocking. The fix would be to move DHCP onto AdGuard itself.

## Installation (Proxmox)

Installed via the [Proxmox VE Community Scripts](https://community-scripts.github.io/ProxmoxVE/):

```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/adguard.sh)"
```

This creates a small Debian LXC (CT 100) with AdGuard Home pre-installed and configured to start on boot.
