# Linux System Administration Basics

A practical reference for getting comfortable with Linux system administration: gaining root access safely, finding admin commands and config files, reading logs, and inspecting hardware and kernel modules.

## Why Administration Is Separate From Regular Use

Linux keeps day-to-day use and system administration apart on purpose. Regular accounts can't touch things that could break the system or expose other users' data — that job belongs to the **root** user (a.k.a. the **superuser**). Even on a single-user laptop, this separation stays in place as a safety net against typos and careless commands.

Common administrator responsibilities include:

- **Filesystems** — adding storage, changing layout outside home directories, backing up any user's files
- **Software installation** — system-wide packages need root so users can't silently install malicious software
- **User and group accounts** — creating and removing them
- **Network interfaces** — increasingly delegated to regular users via NetworkManager, but still root-controlled at a lower level
- **Servers and services** — web, mail, DNS, file sharing, etc., usually run under dedicated service accounts (like `apache` or `rpc`) rather than root, to limit the blast radius if one is compromised
- **Security** — firewalls, access lists, and monitoring for abuse

## Getting Root Access

You rarely log in directly as root. Instead, pick one of these:

- **`su`** — opens a full root shell (`su -` loads root's environment too). Good for a batch of admin work; you `exit` back to your normal shell when done.
- **`sudo`** — runs a *single* command with root privileges using **your own password**, not root's. This is the default on Ubuntu and Fedora for the first created user.

### Granting `sudo` Access

Root-level sudo permissions are managed in **`/etc/sudoers`**, which should only ever be edited with **`visudo`** — it locks the file and validates your syntax before saving:

```bash
visudo
```

Give a user full root privilege:

```
# Prompts for the user's own password on every sudo command
jane    ALL=(ALL)     ALL

# Same, but never prompts for a password
jane    ALL=(ALL)     NOPASSWD: ALL
```

Once granted, a successful `sudo` password entry is typically cached for a few minutes (RHEL/Fedora) so you aren't re-prompted for every command in that window; Ubuntu can be configured differently via the `passwd_timeout` setting.

> **Tip:** `sudo` is more auditable than `su` — it logs *which* user ran an administrative command, whereas `su` only shows that someone with the root password logged in.

## Where Admin Commands, Configs, and Logs Live

### Commands

- **`/usr/sbin`** — administrative and daemon commands (historically split with `/sbin`, now usually merged and symlinked). Look for daemon names ending in `d`, like `sshd` or `cupsd`.
- **`/bin`, `/usr/bin`** — commands regular users can run, sometimes with restricted options for non-root users (e.g., anyone can list mounts with `mount`, but only root can actually mount a filesystem).
- **`/usr/local/bin`, `/usr/local/sbin`** — the right place for commands you add yourself; these directories are typically searched *before* the standard ones, so local versions can override system defaults.
- **Section 8 man pages** (`man 8 <command>`, stored under `/usr/share/man/man8`) document admin-focused commands specifically.

### Configuration Files

Almost everything you configure system-wide — networking, users, services, mail — lives under **`/etc`**. Learning the standard config file layout pays off because it stays consistent across distributions even as GUI tools change.

### Log Files

Modern systemd-based distributions centralize logging through the **systemd journal**, queried with:

```bash
# Follow logs live, useful when plugging in hardware or debugging a service
journalctl -f

# Show only boot-related messages
journalctl -b
```

Traditional plain-text logs are still common under **`/var/log`** (e.g., `messages`, `boot.log`), especially for services that write their own logs outside the journal.

## Inspecting Hardware

A handful of `ls*` and `lspci`/`lscpu`-style commands give you a fast hardware snapshot:

```bash
lscpu          # CPU architecture, core/thread count, cache sizes
lspci          # PCI devices (network cards, GPUs, controllers)
lsusb          # connected USB devices
lsblk          # block devices and how they're mounted
```

Under the hood, the **udev** subsystem dynamically names and manages device nodes as hardware is plugged in or removed, which is why USB drives "just work" without manual configuration.

## Working With Kernel Modules

Modules let the kernel load drivers and features on demand instead of compiling everything in statically.

```bash
# List currently loaded modules
lsmod

# Show details about a specific module (author, description)
modinfo -d e1000
modinfo -a e1000     # author info — handy for reporting bugs upstream

# Load a module temporarily (gone after reboot)
modprobe parport
modprobe parport_pc io=0x3bc irq=auto

# Remove a module that's no longer needed
rmmod parport_pc

# Remove a module *and* any now-unused modules it depended on
modprobe -r parport_pc
```

A module loaded with `modprobe` doesn't survive a reboot — to make it permanent, add the `modprobe` line to a startup script or an appropriate file under `/etc/modules-load.d/`. Built-in modules (compiled directly into the kernel, not loaded dynamically) can't be removed with `rmmod` — you'll get an explicit `ERROR: Module ... is builtin` if you try.

## Quick Reference

| Task | Command |
|---|---|
| Open a root shell | `su -` |
| Run one command as root | `sudo <command>` |
| Edit sudo permissions safely | `visudo` |
| Enable browser-based admin UI | `systemctl enable --now cockpit.socket` |
| Follow system logs live | `journalctl -f` |
| List loaded kernel modules | `lsmod` |
| Load / remove a module | `modprobe <name>` / `rmmod <name>` |
| List PCI / USB / block devices | `lspci` / `lsusb` / `lsblk` |
