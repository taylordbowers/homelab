# Power & Resilience

With the primary node now a laptop, the lab has a built-in UPS: the battery. This page covers how power loss, cold boots, and service failures are handled.

## AC power watchdog (pve)

A small watchdog on `pve` treats the laptop battery as a UPS:

| Condition | Action |
|---|---|
| AC lost, restored within 10 minutes | Nothing (flickers and short outages ride through on battery) |
| AC lost for **10 minutes** | Graceful shutdown of all guests, then the host |
| Battery drops to **25%** (any time on battery) | Graceful shutdown immediately |
| Power-supply sysfs unreadable or garbled | **Nothing.** It fails safe rather than risk a false shutdown |

- Runs as a systemd timer every 30 seconds
- Thresholds live in `/etc/default/ac-power-watchdog`
- `ac-power-watchdog --status` prints the current reading and what it would do
- Sends email alerts through the node's Postfix relay

!!! warning "Test destructive scripts with a CLI flag, not an env/config var"
    An early test run of a similar script powered off a node for real. The `DRY_RUN` variable was overridden by the config file the script sourced. Dry-run belongs in a command-line flag that config can't clobber, tested against a fake shutdown binary on a host you can afford to lose.

The laptop also has **lid-close and sleep disabled** and **charging capped at 80%** to spare the battery from being held at 100% around the clock.

`pve-guide` has no battery. It relies on onboot autostart when power returns.

## Cold-boot ordering

- **GPU containers** wait for a oneshot service that creates the NVIDIA device nodes before Proxmox starts guests. See [GPU Passthrough → Boot ordering](gpu-passthrough.md#boot-ordering).
- **Network mounts** retry: Jellyfin's SMB mount has a retry timer, and Immich's NFS mount is checked before the stack is trusted.
- Every production guest has `onboot: 1`.

## Detection & response

- **[Uptime Kuma](../services/management/uptime-kuma.md)**: 43 monitors, with alerts to Discord (primary) and email
- **[Self-healing](../services/automation/self-healing.md)**: Kuma alerts go to a headless Claude run that can use only an allowlisted set of fixes
- **Weekly health audit**: SMART (ATA + NVMe wear), ZFS health and scrub age, GPU temperature and Xid errors, backup freshness, and a check that every running guest has at least one Kuma monitor
- **Mail relay** on both nodes, so vzdump and system alerts actually get delivered

!!! tip "Postfix: `No worthy mechs found` isn't a bad password"
    If Postfix's SASL auth to Gmail fails with `no mechanism available`, the `libsasl2-modules` package is missing. The credential is fine. If IPv6 has no route, also set `smtp_address_preference = ipv4`.

## Known single points of failure

- **Uptime Kuma and NPM** live on pve-guide (CT 121). If that node is down, monitoring and the reverse proxy go with it. This goes away when the array moves and CT 121 migrates to `pve`.
- **The media dataset has no off-site copy** (see [Backups](backups.md#roadmap)).
