# Linux User Management

This guide provides a comprehensive overview of how to manage user accounts in Linux using the command line. This is particularly useful when desktop GUI tools or web-based interfaces are unavailable, or when performing batch operations via scripts.

## The `useradd` Command

The `useradd` command is the primary tool for creating new user accounts.

### Basic Syntax
    sudo useradd [options] username

> **Note:** You must have root privileges (using `sudo`) to create or modify user accounts.

### Common Options

| Option | Description |
| :--- | :--- |
| `-c "comment"` | Adds a description (e.g., Full Name). Use quotes for multiple words. |
| `-d home_dir` | Sets the custom home directory path. |
| `-e YYYY-MM-DD` | Sets the account expiration date. |
| `-g group` | Assigns the user to a primary group (must exist). |
| `-G group1,group2` | Adds the user to multiple supplementary groups. |
| `-m` | Automatically creates the user's home directory (copies from `/etc/skel`). |
| `-s shell` | Specifies the login shell (e.g., `/bin/bash` or `/bin/tcsh`). |
| `-u UID` | Manually specifies the User ID number. |
| `-o` | Use with ‐u uid to create a user account that has the same UID as another username. |
| `-f -number` | Set the number of days after a password expires until the account is permanently disabled. |
| `-k skel_dir` | Set the skeleton directory containing initial configuration files and login scripts that should be copied to a new user's home directory |
| `‐M` | Do not create the new user's home directory, even if the default behavior is set to create it. |
| `‐p passwd` | Set the password for the account you are adding. This must be an encrypted password. Instead of adding an encrypted password here, you can simply use the passwd user command later to add a password for user. (To generate an encrypted MD5 password, type openssl passwd.)|

---

## Practical Example: Creating a User

To create an account for "Sara Green" (`sara`), run:

    # Add the user with a description
    sudo useradd -c "Sara Green" sara

    # Set the password
    sudo passwd sara

> **Important:** When running `passwd` as root, you can set short or even blank passwords. Regular users cannot do this for themselves.

### Advanced Example

To create a user with a specific primary group, supplementary groups, and a custom shell:

    sudo useradd -g users -G wheel,apache -s /bin/tcsh -c "Sara Green" sara

---

## What Happens Under the Hood?

When you run `useradd`, the system performs several automated tasks:

1. **Defaults:** Reads `/etc/login.defs` and `/etc/default/useradd`.
2. **User Entries:** Creates entries in `/etc/passwd` and `/etc/shadow`.
3. **Groups:** Updates `/etc/group` (usually creating a private group for the user).
4. **Filesystem:** Creates a home directory (usually in `/home/`) and copies default configuration files from `/etc/skel`.

### Understanding `/etc/passwd`
This file stores account information. Each line represents one user:
`sara:x:1002:1007:Sara Green:/home/sara:/bin/tcsh`

* **sara**: Login name.
* **x**: Password placeholder (actual hash is in `/etc/shadow`).
* **1002**: User ID (UID).
* **1007**: Primary Group ID (GID).
* **Sara Green**: Comment/Description.
* ** /home/sara**: Home directory.
* ** /bin/tcsh**: Default login shell.

### Understanding `/etc/group`
This file manages group membership, enabling file sharing access control:
`sara:x:1007:`

* **sara**: Group name.
* **x**: Group password placeholder.
* **1007**: Group ID (GID).
* **(Empty space)**: List of additional users in this group.

### Useradd Default Configuration Options

Use the following options to modify the defaults for user creation. To set these, use the `-D` flag first, followed by the options you wish to change.

#### **-b** `default_home`
Set the default directory in which user home directories are created. 
* **Example:** `-b /garage`
* **Default:** Typically `/home`

#### **-e** `default_expire_date`
Set the default expiration date on which the user account is disabled. 
* **Format:** `YYYY-MM-DD`
* **Example:** `-e 2029-10-17`

#### **-f** `default_inactive`
Set the number of days after a password has expired before the account is disabled. 
* **Example:** `-f 7` (disables the account 7 days after password expiration).

#### **-g** `default_group`
Set the default group in which new users will be placed. 
* **Note:** Normally, `useradd` creates a new group with the same name and ID number as the user.
* **Example:** `-g bears`

#### **-s** `default_shell`
Set the default shell for new users. 
* **Example:** `-s /bin/sh` or `-s /bin/tcsh`

### Usage Example
To update the default home directory location to `/home/everyone` and the default shell to `/bin/tcsh`, run:

`useradd -D -b /home/everyone -s /bin/tcsh`

---

## Modifying Existing Users (`usermod`)

If you need to change account parameters after a user is created, use `usermod`.

| Option | Description |
| :--- | :--- |
| `-c "Name"` | Updates the user's comment/description. |
| `-g group` | Changes the primary group. |
| `-G groups` | Sets new supplementary groups. |
| `-a` | **Crucial:** Use with `-G` to *append* groups instead of replacing them. |
| `-l newname` | Renames the user login. |
| `-d home_dir` | Change the home directory to use for the account. |
| `-e YYYY-MM-DD` | Assign a new expiration date for the account in YYYY‐MM‐DD format. |
| `‐f ‐1` | Change the number of days after a password expires until the account is permanently disabled. |
| `-L / -U` | Locks or Unlocks the user account. You can lock the account by putting an exclamation point at the beginning of
the encrypted password in /etc/shadow. |

*Example: Adding supplementary groups to an existing user:*
    usermod -Ga sales,marketing chris

---

## Deleting Users (`userdel`)

To remove a user, use `userdel`. 

* **Standard Removal:** userdel username
* **Total Removal (Recommended):** userdel -r username
    *(The `-r` flag ensures the home directory is also deleted.)*

---

## Housekeeping: Finding Orphaned Files

Deleting a user (without `-r`) leaves their files on the system with no owner. This is a security risk.

### Find files belonging to a specific user or UID
    find / -user chris -ls
    find / -uid 504 -ls

### Find all orphaned files
To find every file on the system that is not associated with any valid user (a major security concern), run:
    find / -nouser -ls

*Always review orphaned files before assigning them to a new owner or deleting them.*

---

# Understanding Linux Group Accounts

Group accounts are essential for file sharing and managing permissions across multiple users.

## Why Groups are Useful
Groups allow the root user to assign permissions to a collection of users rather than individuals.

**Example: Sales Department Permissions**
If you have a directory `/var/salesstuff/`, you can assign it to the `sales` group.
- The directory permission `rwxrwxr-x` allows the group members to read, write, and execute (enter/list) files.
- Files inside (like `file.txt`) can be restricted or opened for members of the `sales` group via group-level permissions.

## Managing Primary and Supplementary Groups

### Primary Groups
Every user is assigned a **Primary Group**. By default (on Ubuntu, Fedora, RHEL), this is a group created with the same name as the user.
- **Verification:** The Group ID (GID) is found in the third field of the `/etc/passwd` entry.
    - Example: `sara:x:1002:1007:Sara Green:/home/sara:/bin/tcsh` (where `1007` is the GID).
- **Behavior:** Files created by a user are, by default, assigned to their primary group.

### Supplementary Groups
Users can belong to zero or more additional groups (e.g., `sales`, `marketing`).
- These are managed in the `/etc/group` file.
- **Crucial Rule:** Users cannot add themselves to supplementary groups. Only the **root user** can assign users to groups.

### Using Groups Temporarily: `newgrp`
If a user needs to act as a member of a group they belong to (or have permission to join), they can use `newgrp`.

**Example:**
    [sara]$ newgrp sales
    [sara]$ touch file2
    [sara]$ ls -l file2
    -rw-rw-r--. 1 sara sales 0 Jan 18 22:23 file2

*Tip: Root can set a group password (via `gpasswd`) allowing any user to join that group temporarily by providing the password.*

## Creating and Modifying Groups

Only the root user can manage groups using the command line.

### Creating Groups (`groupadd`)
- **Default GID:** `groupadd kings` (assigns the next available ID).
- **Specific GID:** `groupadd -g 1325 jokers` (assigns a specific ID).

*Note: Administrative groups typically use GIDs between 0-999. It is common practice to assign custom groups IDs above 200/1000 to avoid collision with system defaults.*

### Modifying Groups (`groupmod`)
- **Change GID:** `groupmod -g 330 jokers`
- **Rename Group:** `groupmod -n jacks jokers`

### Adding Users to Groups
To assign a group as a supplementary group to an existing user, use the `usermod` command:
    usermod -aG [groupname] [username]

*(Remember the `-a` flag when using `-G` to append the group rather than replacing all existing supplementary groups.)*

---

## Enterprise Linux: Access Control Lists (ACLs)

Standard Linux permissions (rwx) are often too restrictive for enterprise environments because they only allow one owner and one group per file. **Access Control Lists (ACLs)** solve this by allowing you to grant specific permissions to multiple users and groups without changing the file owner.

### Why Use ACLs?
* **Flexibility:** Grant permissions to specific users (e.g., `bill`) or multiple groups (e.g., `sales` and `marketing`) on a single file.
* **Selective Sharing:** Share files without leaving them "wide open" (chmod 777).
* **Inheritance:** Set default ACLs on directories so new files automatically inherit permissions.

### Using `setfacl` and `getfacl`

### Viewing Permissions
When a file has ACLs, a `+` symbol appears in the `ls -l` output:
    -rw-rw-r--+ 1 mary mary 0 Jan 21 09:27 memo.txt

Use `getfacl` to see the full list:
    getfacl /tmp/memo.txt

### Setting Permissions
The `setfacl` command is used to modify (-m) permissions:

* **Grant a user Read/Write access:**
    setfacl -m u:bill:rw filename
* **Grant a group Read/Write access:**
    setfacl -m g:sales:rw filename

### The "Mask" Concept
When you add an ACL, the group permission acts as a **mask**. Even if you grant an ACL user "Read/Write/Execute," the mask limits the maximum permission they can actually use. 

*If the mask is set to 'r--', the ACL user will only be able to read the file, regardless of their own assigned permissions.*

Note: To set mask on acl file, just use "setfacl -m m::permissions filename"

### Default (Inherited) ACLs
To ensure all files created inside a directory automatically share the same permissions, use the **default (d:)** flag:

    # Set default ACL for the 'market' group
    setfacl -m d:g:market:rwx /tmp/mary/

Any new file or sub-directory created inside `/tmp/mary/` will now automatically inherit these settings.

### Enabling ACLs on Filesystems
ACLs must be enabled on the filesystem level. While modern systems (XFS, EXT4) often enable this by default, you may need to add it manually for older or custom partitions.

### Checking Current Support
    # Check current mount options
    mount | grep "/ "
    tune2fs -l /dev/sda1 | grep "mount options"

### Enabling ACLs Permanently
1.  **Edit /etc/fstab:**
    Add `acl` to the options column (the 4th field).
    /dev/sda1  /var/stuff  ext4  defaults,acl  1  2

2.  **Remount without rebooting:**
    mount -o remount /dev/sda1

3.  **Implant in Superblock (alternative):**
    tune2fs -o acl /dev/sda1

Note: mount command will only mount the filesystem temporarily 
---

## Adding Directories for Users to Collaborate

A special set of three permission bits—often ignored by basic `chmod` commands—allows you to set special permissions on commands and directories. These bits are essential for creating collaborative workspaces.

### Special Permission Bits
You can set these bits using `chmod` along with standard read, write, and execute bits.
- **Set user ID bit:** 4 (u+s)
- **Set group ID bit:** 2 (g+s)
- **Sticky bit:** 1 (o+t)

### The Set UID and Set GID Bit Commands (Sidebar)
* **Normal Execution:** When a user runs a command, it runs with that user's permissions.
* **Set UID/GID Execution:** Commands with these bits set run with the permissions of the **owner** (Set UID) or the **group** (Set GID) assigned to the file, rather than the user running the command.
* Example: `su` and `newgrp` use the Set UID bit to gain root permissions (after password verification). You can verify this by checking the `s` in the output of `ls /bin/su`.

---

## Creating Group Collaboration Directories (Set GID)

When you create a Set GID directory, all files created within it are automatically assigned to the group of the directory, not the user's primary group.

### Steps to create a collaborative directory for the "sales" group:

1. **Create the group:**
    groupadd -g 301 sales

2. **Add users (e.g., mary):**
    usermod -aG sales mary

3. **Create the directory:**
    mkdir /mnt/salestools

4. **Assign the group:**
    chgrp sales /mnt/salestools

5. **Set permissions (2775):**
    chmod 2775 /mnt/salestools
    *(This enables Set GID (2), full rwx for user (7), rwx for group (7), and r-x (5) for others.)*

6. **Verify:**
    If mary creates a file in this directory, it will be assigned to the `sales` group, allowing all members of `sales` to read and write to it.

---

## Creating Restricted Deletion Directories (Sticky Bit)

The **sticky bit** creates a restricted deletion directory. While users may have write access to the directory, they **cannot delete files owned by other users**. Only the root user or the owner of the specific file can delete it.

### Example: The /tmp Directory
The `/tmp` directory is a classic restricted deletion directory. Permissions typically appear as:
    drwxrwxrwt. ... /tmp

### Creating a Restricted Deletion Directory
    [mary]$ mkdir /tmp/mystuff
    [mary]$ chmod 1777 /tmp/mystuff
    [mary]$ cp /etc/services /tmp/mystuff/
    [mary]$ chmod 666 /tmp/mystuff/services

With permissions set to `1777`, anyone can write to the `services` file, but only `mary` or `root` can delete the file from the directory.

---

## Centralizing User Accounts

In enterprise environments, managing user accounts locally (`/etc/passwd` and `/etc/shadow`) is impractical. Centralized authentication allows a system to query a server for user credentials.

### Authentication Domains
The `authconfig` command supports several centralized databases:
* **LDAP (Lightweight Directory Access Protocol):** An open standard protocol for directory services.
* **Kerberos:** Provides secure client/server communications using an Authentication Server and Ticket-granting Server. Integrates well with services like ssh and ftp.
* **Winbind:** Used to authenticate Linux users against a Microsoft Active Directory (AD) server.