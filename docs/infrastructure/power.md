# Power & Resilience

With the primary node now a laptop, the lab has a built-in UPS: the battery. This page covers how power loss, cold boots, and service failures are handled.

## AC power watchdog (pve)

A small watchdog on `pve` treats the laptop battery as a UPS:

| Condition | Action |
|---|---|
| AC lost | Nothing. It rides through on battery for as long as the charge lasts (about an hour) |
| Battery drops to **10%** | Graceful shutdown of all guests, then the host |
| Power-supply sysfs unreadable or garbled | **Nothing.** It fails safe rather than risk a false shutdown |

- Runs as a systemd timer every 30 seconds
- Thresholds live in `/etc/default/ac-power-watchdog`
- `ac-power-watchdog --status` prints the current reading and what it would do
- Sends email alerts through the node's Postfix relay

!!! warning "Test destructive scripts with a CLI flag, not an env/config var"
    An early test run of a similar script powered off a node for real. The `DRY_RUN` variable was overridden by the config file the script sourced. Dry-run belongs in a command-line flag that config can't clobber, tested against a fake shutdown binary on a host you can afford to lose.

The laptop also has **lid-close and sleep disabled** and **charging capped at 80%** to spare the battery from being held at 100% around the clock.

The drive array has its own ATX power supply, which has **no** battery. If mains fails, the drives drop while the laptop keeps running, until the watchdog shuts it down. Putting the drive PSU on a small UPS is on the list.

!!! warning "Check the charger after hardware work"
    During the October 2026 drive install, the laptop's barrel plug got nudged loose. The node ran on battery without anyone noticing until the watchdog shut it down 10 minutes later (it had a 10-minute timer then; now it rides until 10% battery). After any work inside the rack, check that `/sys/class/power_supply/AC/online` reads `1`.

## Cold-boot ordering

- **GPU containers** wait for a oneshot service that creates the NVIDIA device nodes before Proxmox starts guests. See [GPU Passthrough → Boot ordering](gpu-passthrough.md#boot-ordering).
- **Mounts:** Jellyfin's SMB mount has a retry timer. Immich and Nextcloud use bind mounts of the local pool, so there's no network race any more.
- Every production guest has `onboot: 1`.

## Detection & response

- **[Uptime Kuma](../services/management/uptime-kuma.md)**: 37 active monitors, with alerts to Discord (primary) and email
- **[Self-healing](../services/automation/self-healing.md)**: Kuma alerts go to a headless Claude run that can use only an allowlisted set of fixes
- **Weekly health audit**: SMART (ATA + NVMe wear), ZFS health and scrub age, GPU temperature and Xid errors, backup freshness, and a check that every running guest has at least one Kuma monitor
- **Mail relay** on the host, so vzdump and system alerts actually get delivered

!!! tip "Postfix: `No worthy mechs found` isn't a bad password"
    If Postfix's SASL auth to Gmail fails with `no mechanism available`, the `libsasl2-modules` package is missing. The credential is fine. If IPv6 has no route, also set `smtp_address_preference = ipv4`.

## Known single points of failure

- **Everything is one machine now.** Since October 2026 every guest, the array and the backups share one laptop and one rack. That's simpler, and it removed every cross-node dependency, but the whole lab now shares one power path and one chassis.
- **Monitoring lives on the thing it monitors.** Uptime Kuma (CT 121) runs on `pve`, so a full host outage silences the alerts too. An external heartbeat would cover it.
- **No off-site copy yet** of either the guests or the media dataset (see [Backups](backups.md#roadmap)).
