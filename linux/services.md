# Starting and Stopping Linux Services (systemd)

A practical guide to managing services on modern Linux systems using **systemd** — how the init system boots your machine, how to control individual services, and how to add your own.

## What an Init System Does

The **init daemon** is the first process the kernel starts (always **PID 1**), and every other process traces back to it. It's responsible for:

- Launching processes needed at boot (login shells, background services)
- Grouping services into sets — traditionally **runlevels**, now **targets**
- Enforcing startup order and dependencies between services
- Starting, stopping, restarting, and reloading services on demand

**systemd** is the dominant init system today (Fedora, RHEL, Ubuntu, and most others), replacing the older **SysVinit**. Its biggest practical advantage is *parallel* startup — SysVinit started services strictly one after another, so a single stuck service could stall the whole boot. systemd only waits on actual dependencies.

You can confirm you're on a systemd system a couple of ways:

```bash
ps -e | head -1          # PID 1 will show something under systemd
ls -l /sbin/init         # often just a symlink to systemd itself
```

## Units: systemd's Building Blocks

systemd manages more than just services — everything it controls is a **unit**. The two you'll deal with most:

- **`.service`** units — manage a single daemon (e.g., `sshd.service`)
- **`.target`** units — a named grouping of other units (e.g., `multi-user.target`), roughly equivalent to old-style runlevels

List active service units:

```bash
systemctl list-units | grep .service
```

### Reading a Service Unit File

```bash
cat /lib/systemd/system/sshd.service
```

```ini
[Unit]
Description=OpenSSH server daemon
After=network.target sshd-keygen.target

[Service]
Type=notify
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Key fields worth knowing:

- **`After`** — ordering only; doesn't force this unit to run, just says what should start first if both are starting
- **`ExecStart` / `ExecReload`** — the actual commands used to start or reload the daemon
- **`WantedBy`** — which target "pulls in" this service when that target activates

### Wants vs. Requires

A target's **Wants** list are units it tries to start, but failure isn't fatal to the target. **Requires** is much stricter — if a required unit fails, the whole target is considered failed too.

```bash
# See everything multi-user.target wants to start (pipe to less, it's long)
systemctl show --property "Wants" multi-user.target | less

# See its hard requirements (much shorter list)
systemctl show --property "Requires" multi-user.target
```

## Controlling Services

### Immediate Control (does not survive a reboot)

```bash
systemctl start   sshd.service     # start now
systemctl stop    sshd.service     # stop now
systemctl restart sshd.service     # stop, then start
systemctl reload  sshd.service     # re-read config without stopping the process
systemctl status  sshd.service     # check current state, recent log lines
```

**Restart vs. reload** matters on a busy server: `restart` kills and relaunches the process (dropping in-flight connections), while `reload` just tells the running daemon to re-read its config — much less disruptive when supported.

If you edit a unit file directly, apply the change with:

```bash
systemctl daemon-reload
```

### Persistent Control (survives a reboot)

`start`/`stop` only affect the *current* boot session. To control whether a service comes up automatically next time the machine boots, use **enable**/**disable**:

```bash
systemctl enable  sshd.service     # start automatically at boot
systemctl disable sshd.service     # don't start at boot (doesn't stop it now)
```

Enabling/disabling just manages symlinks under `/etc/systemd/system/*.wants/` pointing back to the real unit file — but always use `systemctl` rather than creating those symlinks by hand.

Some services are **static** — always enabled, can't be disabled (e.g., `dbus.service`), because other services depend on them unconditionally.

To make absolutely sure a service can never start — even as a dependency of something else — **mask** it:

```bash
systemctl mask   MyService.service   # links the unit to /dev/null
systemctl unmask MyService.service   # reverse it
```

## Setting the Default Target (Runlevel)

The boot-time target is controlled by a symlink at `/etc/systemd/system/default.target`. Common targets:

| Target | Roughly equivalent to |
|---|---|
| `multi-user.target` | text-mode, networked (old runlevel 3) |
| `graphical.target` | full desktop (old runlevel 5) |
| `rescue.target` | single-user/rescue mode (old runlevel 1) |

```bash
systemctl get-default
systemctl set-default graphical.target
```

Legacy runlevel numbers still work as a compatibility shim — `runlevel0.target` through `runlevel6.target` are symlinked to their systemd equivalents, and the `runlevel` command shows the previous and current numeric runlevel for anyone used to the old scheme.

## Adding a Custom Service

Three steps get a new daemon under systemd's control:

**1. Write a unit file.** At minimum you need `Description` and `ExecStart`:

```ini
# MyNewService.service
[Unit]
Description=My New Service

[Service]
ExecStart=/usr/bin/my-new-service

[Install]
WantedBy=multi-user.target
```

**2. Put it in the right place: `/etc/systemd/system/`.**

This directory is for local customizations — files here are never overwritten by package upgrades, and they take priority over any same-named file under `/lib/systemd/system/` (which *is* managed by packages).

```bash
cp MyNewService.service /etc/systemd/system/
systemctl daemon-reload
```

**3. Link it into a target's Wants directory** so it starts automatically:

```bash
ln -s /etc/systemd/system/MyNewService.service \
      /etc/systemd/system/multi-user.target.wants/MyNewService.service
```

(This step is optional — skip it if you only ever want to start the service manually.) To start it immediately without waiting for a reboot:

```bash
systemctl start MyNewService.service
```

## Quick Reference

| Task | Command |
|---|---|
| Check what init system you're on | `ps -e \| head -1` |
| Start / stop right now | `systemctl start\|stop <unit>` |
| Reload config without downtime | `systemctl reload <unit>` |
| Start automatically at boot | `systemctl enable <unit>` |
| Prevent a service from *ever* starting | `systemctl mask <unit>` |
| Check / change the default boot target | `systemctl get-default` / `set-default` |
| Apply an edited unit file | `systemctl daemon-reload` |