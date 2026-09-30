# Tailscale

WireGuard-based mesh VPN for remote access to the whole lab, with no inbound ports opened for it.

## Setup

- **Host:** the `pve` node itself (host install, not a container)
- **Roles:**
    - **Subnet router:** advertises the home LAN (`10.0.0.0/24`), so any tailnet device can reach every guest by its LAN IP
    - **Exit node:** tailnet devices can route all their internet traffic through home, which is handy on untrusted Wi-Fi
- **Also on the tailnet:** the AMP game-server VM

## Why on the host?

Running Tailscale on the Proxmox host means remote access doesn't depend on any guest being up. If every container is down, you can still reach the Proxmox UI and SSH remotely to fix things.

## Setup notes

```bash
curl -fsSL https://tailscale.com/install.sh | sh

# Enable forwarding for subnet routing / exit node
echo 'net.ipv4.ip_forward = 1' > /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' >> /etc/sysctl.d/99-tailscale.conf
sysctl -p /etc/sysctl.d/99-tailscale.conf

tailscale up --advertise-routes=10.0.0.0/24 --advertise-exit-node
# then approve the route + exit node in the Tailscale admin console
```

!!! tip "UDP GRO forwarding"
    Tailscale suggests enabling UDP GRO forwarding on the physical NIC for better subnet-router throughput (`ethtool -K <nic> rx-udp-gro-forwarding on rx-gro-list off`). That setting **doesn't survive a reboot**, so make it persistent with a small systemd unit or a `post-up` line on the interface.

## History

Tailscale originally ran on the old `pve2` node and was rebuilt on `pve` during the July 2026 migration. The subnet route and exit node were both re-verified after the rebuild.
