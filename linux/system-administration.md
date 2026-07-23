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


## Remote System Administration with SSH

### SSH Core Ecosystem Overview
Secure Shell (SSH) is the foundation of remote Linux management. By default, SSH authenticates via standard Linux user accounts using encrypted network traffic.

| Tool | Primary Purpose | Connection Type |
| :--- | :--- | :--- |
| `ssh` | Remote login shell & remote command execution | Encrypted Interactive / Batch |
| `scp` | Simple secure file copy | Non-interactive Batch |
| `rsync` | Efficient incremental file synchronization & backup | Encrypted Delta Sync |
| `sftp` | Interactive file management | Interactive File Session |

---

### Interactive Remote Logins (`ssh`)
To log into a remote system running the `sshd` daemon, pass the user account and host IP or hostname:

```bash
$ ssh johndoe@10.140.67.23
```

#### First-Time Host Verification
Upon connecting to a host for the first time, SSH requests public key verification:

```text
The authenticity of host '10.140.67.23 (10.140.67.23)' can't be established.
RSA key fingerprint is a4:28:03:85:89:6d:08:fa:99:15:ed:fb:b0:67:55:89.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.140.67.23' (RSA) to the list of known hosts.
johndoe@10.140.67.23's password: *********
```

- Typing `yes` downloads the host’s public key into `~/.ssh/known_hosts`.
- Server-side key pairs reside in `/etc/ssh/`.

#### Ending Sessions
- End shell session cleanly: `exit` or `logout`.
- Unresponsive sessions can be forced closed using the escape sequence: `~.`

> **Caution: Host Key Changes**  
> If a remote operating system is reinstalled, its host key changes. Subsequent connections fail with warnings. To resolve legitimate changes, delete the offending key line from `~/.ssh/known_hosts` and reconnect.

---

### Remote Execution & X11 Graphical Forwarding

#### Non-Interactive Command Execution
Pass the command directly after the connection parameters:

```bash
$ ssh johndoe@10.140.67.23 hostname
host01.example.com
```

When passing arguments or relative paths, enclose the entire remote command in quotes:

```bash
$ ssh johndoe@10.140.67.23 "cat myfile"
```

### X11 Graphical Forwarding (`ssh -Y`)
To stream remote GUI utilities locally, ensure `X11Forwarding yes` is set in `/etc/ssh/sshd_config` on the server, then initiate a connection with the `-Y` flag:

```bash
# Execute single remote GUI utility locally
$ ssh -Y johndoe@10.140.67.23 xterm

# Execute multiple GUI apps from an interactive shell in the background
$ ssh -Y johndoe@10.140.67.23
$ xterm &
$ xeyes &
$ exit
```

### File Transfer Protocols (`scp` vs. `rsync`)

#### Basic Copying with `scp`
Transfer files securely between local and remote systems:

```bash
$ scp /etc/services johndoe@10.140.67.23:/tmp
```

#### Recursive Directory Copying (`scp -r`)
To copy entire directory structures recursively:

```bash
$ scp -r johndoe@10.140.67.23:/usr/share/man/man1/ /tmp/
```

#### High-Performance Synchronization (`rsync`)
Unlike `scp`, `rsync` uses delta transfer algorithms to only copy modified files, while preserving file ownership, timestamps, and symbolic links using the archive (`-a`), verbose (`-v`), and symlink (`-l`) flags:

```bash
$ rsync -avl johndoe@10.140.67.23:/usr/share/man/man1/ /tmp/
```

#### Exact State Mirroring (`rsync --delete`)
While `scp` and standard `rsync` only add or overwrite files on the destination, passing the `--delete` flag forces the destination to become an exact mirror of the source. It achieves this by automatically deleting any files in the destination directory that no longer exist in the source directory:

```bash
$ rsync -av --delete /var/www/html/ johndoe@10.140.67.23:/var/www/backup/
```

---

### Interactive File Browsing (`sftp`)
Establish an interactive SSH File Transfer Protocol session:

```bash
$ sftp johndoe@10.140.67.23
Connected to 10.140.67.23.
sftp> help
sftp> ls
sftp> cd /var/log
sftp> get messages /tmp/messages.local
Fetching /var/log/messages to /tmp/messages.local
sftp> put /tmp/localfile.txt /tmp/remotefile.txt
Uploading /tmp/localfile.txt to /tmp/remotefile.txt
sftp> bye
```

---

###  Key-Based Passwordless Authentication

To configure secure key-based authentication without requiring password entries:

#### Step 1: Generate SSH Key Pairs Locally
On your local administrative machine, generate an Ed25519 or RSA key pair:

```bash
$ ssh-keygen -t ed25519 -C "admin@example.com"
```
*(Accept the default path `~/.ssh/id_ed25519`)*

#### Step 2: Copy Public Key to Remote Host
Install the public key into the target user's `~/.ssh/authorized_keys` file automatically:

```bash
$ ssh-copy-id -i ~/.ssh/id_ed25519.pub johndoe@10.140.67.23
```

#### Step 3: Verify Passwordless Login
Connect to the host—you should gain access immediately without a password prompt:

```bash
$ ssh johndoe@10.140.67.23
```

---

### Hardening SSH Security
Edit `/etc/ssh/sshd_config` on the remote server to enforce strict security baselines:

```text
# Disable root SSH access
PermitRootLogin no

# Disable password authentication (forces public keys)
PasswordAuthentication no

# Restrict SSH port from default 22
Port 2222
```

After modifying settings, restart the daemon:

```bash
$ sudo systemctl restart sshd
```

---

## Storage & Filesystem Health Monitoring

### Filesystem Space Assessment (`df`)
Check storage utilization across mounted filesystems:

```bash
# Display space in human-readable format (MB, GB)
$ df -h

# Include filesystem types (ext4, xfs, etc.)
$ df -hT
```

---

### Directory Usage Profiling (`du`)
Analyze directory size footprints to locate space-hogging folders:

```bash
# Estimate space used by a specific directory tree
$ du -sh /var/log/

# Summarize subdirectories one level deep
$ du -h --max-depth=1 /var/
```

---

### Deep Storage Auditing & Cleanup (`find`)
Locate and manage large or old files dynamically across filesystems:

```bash
# Find files larger than 100MB in /var
$ find /var -type f -size +100M

# Find files modified more than 30 days ago
$ find /var/log -type f -mtime +30

# Locate files >100MB and prompt before deletion
$ find /tmp -type f -size +100M -ok rm {} \;
```

## System Logging, Auditing & Journal Analysis

### Linux Logging Fundamentals & `/var/log`
The `/var/log` directory is the central repository for system and service logs. Depending on your distribution (RHEL/CentOS vs. Debian/Ubuntu), key logs include:

- `/var/log/messages` or `/var/log/syslog`: Global system messages and non-critical service logs.
- `/var/log/secure` or `/var/log/auth.log`: Authentication and authorization logs (SSH logins, `sudo` usage).
- `/var/log/dmesg`: Kernel ring buffer logs (hardware detection and driver loading during boot).

---

### Searching & Real-Time Monitoring (`grep`, `tail`)
For traditional text-based logs, standard utilities provide rapid triage capabilities:

```bash
# Watch a log file in real-time as new events are appended
$ tail -f /var/log/secure

# Search for specific error patterns (case-insensitive)
$ grep -i "failed password" /var/log/secure

# Combine commands to filter a live log stream
$ tail -f /var/log/messages | grep -i "error"
```

---

### Modern Log Analysis with `journalctl`
Systems utilizing `systemd` collect logs in a binary format managed by `systemd-journald`. Use `journalctl` to query this database:

```bash
# View all logs starting from the current boot
$ journalctl -b

# Follow new journal entries in real-time (similar to tail -f)
$ journalctl -f

# Filter logs for a specific service (e.g., SSH daemon)
$ journalctl -u sshd

# View detailed logs from a specific timeframe
$ journalctl --since "2026-07-23 08:00:00" --until "2026-07-24 01:00:00"

# Jump directly to the end of the journal with extra details for troubleshooting
$ journalctl -xe
```

---

### User Login & Access Auditing (`last`, `lastlog`)
Track which users have accessed the system and when:

```bash
# Display recent login sessions (reads /var/log/wtmp)
$ last
johndoe  pts/0        10.140.67.23    Fri Jul 24 01:15   still logged in
root     tty1                         Thu Jul 23 22:10 - 23:45  (01:35)

# Display the most recent login time for all system users (reads /var/log/lastlog)
$ lastlog
```

---

## Module 4: Linux User & Group Management

### 4.1 Core Identity Files
User and group configurations are managed locally through three critical text files:

- `/etc/passwd`: Stores basic user account information.
  - Format: `username:x:UID:GID:Comment:HomeDir:LoginShell`
- `/etc/shadow`: Stores secure, encrypted user passwords and password aging policies. 
  - Format: `username:$6$hashedpassword:last_changed:min:max:warn:inactive:expire:`
- `/etc/group`: Stores group information and membership.
  - Format: `group_name:x:GID:user_list`