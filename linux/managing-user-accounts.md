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