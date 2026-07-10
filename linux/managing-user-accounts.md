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
| `-m` | Creates the user's home directory (copies from `/etc/skel`). |
| `-s shell` | Specifies the login shell (e.g., `/bin/bash` or `/bin/tcsh`). |
| `-u UID` | Manually specifies the User ID number. |

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

## Modifying Existing Users (`usermod`)

If you need to change account parameters after a user is created, use `usermod`.

| Option | Description |
| :--- | :--- |
| `-c "Name"` | Updates the user's comment/description. |
| `-g group` | Changes the primary group. |
| `-G groups` | Sets new supplementary groups. |
| `-a` | **Crucial:** Use with `-G` to *append* groups instead of replacing them. |
| `-l newname` | Renames the user login. |
| `-L / -U` | Locks or Unlocks the user account. |

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