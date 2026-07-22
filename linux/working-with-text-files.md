# The Ultimate Linux Text Editing, File Discovery, & Content Filtering Guide

---

## Part 1: Text Editing with `vi` and `vim`

The `vi` editor (and its modern upgrade, `vim`) is an incredibly powerful, terminal-based text editor available by default on almost every Linux distribution. The secret to mastering Vim is understanding its **modes**.

### 1. Understanding Vim's Modes: The "Gear Shift" Concept
Vim operates using specific modes. You must switch to the appropriate mode depending on whether you want to navigate, type text, or run configuration commands.

* **Command Mode:** The default mode upon opening a file. Typing here runs shortcuts for navigation, deletion, copying, and pasting.
* **Insert Mode:** The mode used for typing text directly into the file.
* **Ex Mode (Command-Line Mode):** Triggered by typing `:` while in Command Mode, this mode allows you to execute environment configurations, save, or exit.

> ⚠️ **The Golden Rule:** Pressing the **`Esc`** key (sometimes twice if you are deep within an Ex command) instantly drops you out of any active mode and returns you safely to **Command Mode**. 

#### Opening a File
To open or create a file from the terminal, pass the file path to the editor:
```bash
$ vi /tmp/test
```

---

### 2. Entering Insert Mode (Adding Text)
To type text, you must transition from Command Mode to Insert Mode. Use one of the following commands based on where you want the cursor to begin:

| Command | Action Name | Where Text Begins |
| :---: | :--- | :--- |
| **`i`** | **Insert** | Directly **to the left** (before) the cursor. |
| **`I`** | **Insert at Beginning** | At the **start of the current line**. |
| **`a`** | **Add (Append)** | Directly **to the right** (after) the cursor. |
| **`A`** | **Add at End** | At the **end of the current line**. |
| **`o`** | **Open Below** | Opens a **new blank line below** the current line. |
| **`O`** | **Open Above** | Opens a **new blank line above** the current line. |

---

### 3. Navigating the File
Vim allows complete file navigation directly from the home row of the keyboard, eliminating the need to move your hands to the standard arrow keys.

#### Basic Directional Movement
* **`h`** $\rightarrow$ Moves **Left** one character.
* **`j`** $\rightarrow$ Moves **Down** one line.
* **`k`** $\rightarrow$ Moves **Up** one line.
* **`l`** $\rightarrow$ Moves **Right** one character.
* *(Alternative: `Backspace` moves left; `Spacebar` moves right).*

#### Advanced Horizontal & Screen Movement
* **`w`** $\rightarrow$ Move to the start of the **next word** (words are delimited by spaces, tabs, or punctuation).
* **`W`** $\rightarrow$ Move to the start of the **next word** (delimited strictly by spaces or tabs).
* **`b`** $\rightarrow$ Move to the start of the **previous word** (delimited by spaces, tabs, or punctuation).
* **`B`** $\rightarrow$ Move to the start of the **previous word** (delimited strictly by spaces or tabs).
* **`0` (Zero)** $\rightarrow$ Jump to the **beginning** of the current line.
* **`$`** $\rightarrow$ Jump to the **end** of the current line.
* **`H` (High)** $\rightarrow$ Jump to the **top-left corner** of the visible screen (first line on screen).
* **`M` (Middle)** $\rightarrow$ Jump to the first character of the **middle line** on the screen.
* **`L` (Low)** $\rightarrow$ Jump to the **bottom-left corner** of the visible screen (last line on screen).

#### Scrolling Through Large Files
* **`Ctrl + f`** $\rightarrow$ Page **Forward** (Down) one full screen.
* **`Ctrl + b`** $\rightarrow$ Page **Backward** (Up) one full screen.
* **`Ctrl + d`** $\rightarrow$ Page **Down** one-half screen.
* **`Ctrl + u`** $\rightarrow$ Page **Up** one-half screen.
* **`G`** $\rightarrow$ Jump directly to the **last line** of the file.
* **`1G`** $\rightarrow$ Jump directly to the **first line** of the file.
* **`[Number]G`** $\rightarrow$ Jump directly to a specific line number (e.g., `35G` navigates to line 35).

    *Note: To show the number line in open vim files, just add set number in a file located at ~/.vimrc. After then, opening a file in vim will showcase the number line along with text.*

---

### 4. The Vim Grammar: Actions, Movements, and Multipliers
Editing commands in Vim are combinable. You can chain a **Multiplier (Number)**, an **Action Key**, and a **Movement Key** together to execute highly precise edits.

#### Core Action Keys
* **`x`** $\rightarrow$ Deletes the single character under the cursor.
* **`X`** $\rightarrow$ Deletes the single character directly before the cursor.
* **`d`** $\rightarrow$ **Delete** (cuts selected text into the buffer).
* **`c`** $\rightarrow$ **Change** (erases selected text and instantly switches to *Insert Mode*).
* **`y`** $\rightarrow$ **Yank** (copies text into the buffer).

#### 💡 The Formula: `[Number] + [Action] + [Movement]`

**Examples of Action + Movement:**
* **`dw`** $\rightarrow$ **D**elete from the current cursor position to the start of the next **w**ord.
* **`db`** $\rightarrow$ **D**elete from the cursor back to the start of the previous **w**ord.
* **`dd`** $\rightarrow$ **D**elete the entire current line.
* **`yy`** $\rightarrow$ **Y**ank (copy) the entire current line into the buffer.
* **`c$`** $\rightarrow$ **C**hange text from the cursor to the **end of the line**.
* **`c0`** $\rightarrow$ **C**hange text from the cursor back to the **beginning of the line**.
* **`cl`** $\rightarrow$ **C**hange the current **letter** under the cursor.
* **`cc`** $\rightarrow$ **C**hange the entire current line.
* **`y)`** $\rightarrow$ **Y**ank text from the cursor to the end of the current sentence.
* **`y}`** $\rightarrow$ **Y**ank text from the cursor to the end of the current paragraph.

**Examples using Multipliers:**
* **`3dd`** $\rightarrow$ **Delete 3 lines** starting from the current line.
* **`3dw`** $\rightarrow$ **Delete the next 3 words**.
* **`5cl`** $\rightarrow$ **Change the next 5 letters** (removes the letters and enters Insert Mode).
* **`12j`** $\rightarrow$ **Move down 12 lines**.
* **`5cw`** $\rightarrow$ **Change the next 5 words**.
* **`4y)`** $\rightarrow$ **Yank the next 4 sentences**.

#### Pasting (Putting) Text
Text that has been stored in the buffer via deletion, changing, or yanking can be reinserted using the following:
* **`p` (lowercase):** Pastes text **to the right** of the cursor (for words) or **below** the current line (for whole lines).
* **`P` (uppercase):** Pastes text **to the left** of the cursor (for words) or **above** the current line (for whole lines).

#### The Magic Repeat Command (`.`)
The period key (**`.`**) repeats your most recent editing action (such as a deletion, change, or paste). 

> **Example:** To change multiple occurrences of a word like "Joe" to "Jim":
> 1. Position the cursor at the start of the first instance of "Joe".
> 2. Type `cw`, type `Jim`, and press `Esc`.
> 3. Use search commands to locate the next instance of "Joe".
> 4. Press **`.`** to execute the exact same substitution instantly.

---

### 5. Searching for Text
* **`/pattern`** $\rightarrow$ Searches **forward** through the file for the specified pattern string.
    * *Example:* `/hello` finds the next instance of "hello".
    * *Example with Wildcards:* `/The.*foot` finds a line containing "The", followed by any combination of characters, followed by "foot".
* **`?pattern`** $\rightarrow$ Searches **backward** through the file for the specified pattern string.
    * *Example:* `?[pP]rint` searches backward for either "print" or "Print".
* **Navigation:** Press **`n`** to move to the next occurrence in the same direction, or **`N`** to move to the next occurrence in the opposite direction.

---

### 6. Using Ex Mode (Command-Line Operations)
Type a colon (**`:`**) while in Command Mode to access the bottom command line for global search and substitution commands.

* **`:g/Local`** $\rightarrow$ Searches for the word "Local" and prints every line containing it.
* **`:s/Local/Remote`** $\rightarrow$ Substitutes "Remote" for the *first* occurrence of "Local" on the *current line* only.
* **`:g/Local/s//Remote`** $\rightarrow$ Substitutes the first occurrence of "Local" on **every line** of the file with "Remote".
* **`:g/Local/s//Remote/g`** $\rightarrow$ Substitutes **every occurrence** of "Local" with "Remote" across the **entire file**.
* **`:g/Local/s//Remote/gp`** $\rightarrow$ Substitutes every occurrence of "Local" with "Remote" globally and prints each modified line to verify the changes.

---

### 7. Saving, Exiting, and Recovery Operations
Ensure you are in Command Mode (`Esc`) before issuing exit instructions:

* **`ZZ`** or **`:wq`** $\rightarrow$ **W**rite (Save) all outstanding changes and **q**uit the editor.
* **`:w`** $\rightarrow$ **W**rite (Save) the file to disk while keeping the editing session open.
* **`:q`** $\rightarrow$ **Q**uit the file (fails if there are unsaved changes).
* **`:q!`** $\rightarrow$ **Force quit and abandon changes**. Restores the file back to its state at the most recently saved version.

#### Practical System Survival Tips
* **Undo:** Press **`u`** to undo the previous change. Press it repeatedly to roll back alterations consecutively all the way back to the beginning of the editing session.
* **Redo:** Press **`Ctrl + r`** to reverse the last undo command (Redo).
* **Caps Lock Warning:** Vim commands are strict regarding case sensitivity. If `Caps Lock` is left on accidentally, keystrokes will execute unintended commands without warning.
* **File Details (`Ctrl + g`):** Press `Ctrl + g` to display the active filename, current line position, total lines, percentage through the document, and active column at the bottom of the display.
* **Running Shell Commands (`:!command`):** Execute a local terminal command without shutting down your editor session.
    * *Example:* `:!date` outputs the system time.
    * *Example:* `:!pwd` lists the working directory path.
    * *Example:* `:!bash` drops into a nested shell environment (type `exit` to return directly to Vim). *Note: Run `:w` to save changes before executing an external shell.*

---

## Part 2: Finding Files in Linux

### 1. Using `locate` / `plocate` (Database Search)
Linux systems utilize a background utility named `updatedb` (often run automatically once a day) to index all file paths into a system database. The `locate` command (replaced by `plocate` on newer systems via a symbolic link) quickly queries this database.

#### Performance Trade-Offs
* **Advantage:** Extremely fast because it scans a single index database rather than running live drive traversals.
* **Disadvantage:** Cannot locate newly created files added to the filesystem *after* the most recent `updatedb` execution.

#### Manual Database Refresh
To force the indexing database to include up-to-the-moment file paths, run `updatedb` from a root shell:
```bash
# updatedb
```

#### Understanding Index Pruning
The contents of `/etc/updatedb.conf` dictate paths and filesystems that are excluded from database collection. This prunes network storage layouts (like `nfs` or `cifs`), optical drives (`iso9660`), and transient folders (`/tmp`, `/var/spool`).

Example configuration values:
```text
PRUNEFS = "9p afs autofs binfmt_misc ceph cgroup cgroup2 cifs nfs nfs4 proc ramfs sysfs tmpfs udev"
PRUNEPATHS = "/tmp /media /dev /sys /proc /run /var/cache /var/spool"
```

#### Practical Examples
* **Standard Filename Query:**
```bash
$ locate .bashrc
/etc/skel/.bashrc
/home/cnegus/.bashrc
```
    *Note: Standard user permissions apply. A regular user cannot see paths via `locate` that they lack rights to see via standard directory browsing (e.g., `/root`). Running as root displays comprehensive global system results.*
* **Case-Insensitive Searching (`-i`):**
```bash
$ locate -i dir_color
/etc/DIR_COLORS
/usr/share/man/man5/dir_colors.5.gz
```
* **String Matching Behaviors:** `locate` scans the entire absolute path string. Searching for a term returns any path containing that substring anywhere in its tree structure:
```bash
$ locate bzip2
/usr/bin/bzip2
/var/lib/dpkg/info/bzip2.list
```

---

### 2. Live File Searching with `find`
The `find` command searches the active filesystem structure in real-time. It is slower than database lookups but provides an accurate snapshot of the current state of the filesystem, filtered by a wide variety of attributes.

#### Basic Syntax Framework
```bash
$ find [Starting_Directory] [Filters/Options]
```
* Running `find` with no parameters lists all elements under the current directory.
* Specifying a path (e.g., `find /etc`) limits the traversal parameters exclusively to that sub-tree.

> 💡 **Tip: Silencing Access Warnings**
> Scanning system paths as an unprivileged user triggers numerous "Permission denied" errors. Append **`2> /dev/null`** to redirect standard errors away from your display output.

#### Filtering and Common Selections

##### A. Name Matching (`-name` / `-iname`)
* **Case-Sensitive Exact Match:**
```bash
# find /etc -name passwd
/etc/passwd
```
* **Case-Insensitive Wildcard Match (`-iname`):**
```bash
# find /etc -iname '*passwd*'
/etc/security/opasswd
/etc/pam.d/chpasswd
```

##### B. Sizing Filters (`-size`)
Prefix numerical arguments with `+` for greater than, or `-` for less than.
* **Find files strictly larger than 10MB:**
```bash
$ find /usr/share/ -size +10M
```
* **Find files strictly smaller than 1MB:**
```bash
$ find /mostlybig -size -1M
```

##### C. Ownership Filters (`-user` / `-group`)
* **Isolate items owned by a particular user profile:**
```bash
$ find /home -user chris -ls
```
    *(The `-ls` parameter formats matches into a detailed breakdown mirroring `ls -l` format output).*
* **Isolate items belonging to a particular system group:**
```bash
# find /etc -group lp -ls
```

##### D. Time and Date Filters
Matches files based on access history (`a`), content modifications (`m`), or structural inode metadata updates (`c`). Criteria parameters accept **days** (`time`) or **minutes** (`min`). Prefix numbers with `-` for "within X time" and `+` for "older than X time".

| Option | Match Condition |
| :--- | :--- |
| **`-mmin -10`** | Content modified within the past 10 minutes. |
| **`-ctime -3`** | Metadata status changed within the last 3 days. |
| **`-atime +300`** | File has not been accessed for more than 300 days. |

* **Example (Check configuration alterations over the last 10 minutes):**
```bash
$ find /etc/ -mmin -10
```
* **Example (Check system binary status shifts over the past 3 days):**
```bash
$ find /bin /usr/bin /sbin /usr/sbin -ctime -3
```

##### E. Mode Permission Filters (`-perm`)
Permissions match exactly by default, minimally via a hyphen prefix (`-`), or broadly via a logical slash prefix (`/`). 

* **Exact Value Match:** Returns items matching the permission profile exactly.
```bash
$ find /usr/bin -perm 755 -ls
```
* **Minimum Match Guard (`-`):** Returns items where *at least* all specified bits are set. This example isolates directories (`-type d`) writable by User, Group, and Other.
```bash
$ find /home/chris/ -perm -222 -type d -ls
```
* **Logical Split Match (`/`):** Returns items where *any* of the defined write bits are enabled.
```bash
$ find /myreadonly -perm /222 -type f
```
* **Isolating Public Risks:** Locate regular files (`-type f`) where write parameters are open to "other" system users, irrespective of how owner or group parameters are configured.
```bash
$ find . -perm -002 -type f -ls
```

---

### 3. Combining Logic Conditions (`-and`, `-or`, `-not`)
Construct targeted search structures using conditional logical gates.

* **Logical OR (`-o` / `-or`):** Matches either condition. Wrap multiple parameters in escaped parentheses `\(` `\)` to ensure proper command interpretation.
```bash
$ find /var/allusers \( -user joe -o -user chris \) -ls
```
* **Logical NOT (`-not`):** Inverts the matching logic filter.
```bash
$ find /var/allusers/ -user joe -not -group joe -ls
```
* **Logical AND (`-and`):** Matches only when both conditions are true.
```bash
$ find /var/allusers/ -user joe -and -size +1M -ls
```

---

### 4. Executing Commands on Found Files
You can perform automated tasks on the items identified by `find` using the `-exec` or `-ok` actions.

#### Core Execution Syntax
```bash
$ find [options] -exec command {} \;
$ find [options] -ok command {} \;
```
* **`{}` (Curly Braces):** A dynamic placeholder variable replaced by the path of each found file.
* **`\;` (Escaped Semicolon):** Terminates the execution string passed to the command argument.

#### Non-Interactive (`-exec`) vs. Interactive (`-ok`)
* **`-exec`:** Executes the command on every matched target immediately without prompting.
* **`-ok`:** Prompts the user for confirmation before executing the command on each individual file. **Always use `-ok` for destructive tasks like moving (`mv`) or deleting (`rm`) files.**

#### Practical Execution Examples
* **Passing filenames into string echoes:**
```bash
$ find /etc -iname passwd -exec echo "I found {}" \;
I found /etc/pam.d/passwd
I found /etc/passwd
```
* **Tracking large assets and piping to an external sort order tool:**
```bash
$ find /usr/share -size +5M -exec du {} \; | sort -nr
26656 /usr/share/fonts/opentype/noto/NotoSerifCJK-Bold.ttc
```
* **Staging user data with an interactive safety confirmation prompt:**
```bash
# mkdir /tmp/joe
# find /var/allusers/ -user joe -ok mv {} /tmp/joe/ \;
< mv … /var/allusers/dict.dat> ? y
```

---

## Part 3: Content Filtering inside Files with `grep`

The **`grep`** command scans **inside text files** to isolate and display lines that match a specified text string or pattern.

### 1. File Context Searching
By default, `grep` searches for text using strict case sensitivity.
* **Standard Matching:**
```bash
$ grep dns /etc/services
mdns 353/udp # Multicast DNS
```
* **Case-Insensitive Matching (`-i`):** Instructs `grep` to ignore character casing.
```bash
$ grep -i dns /etc/services
domain-s 853/tcp # DNS over TLS [RFC7858]
mdns 353/udp # Multicast DNS
```

---

### 2. Advanced Output Modifiers
* **Invert Matching (`-v`):** Outputs all lines **except** those containing the matched pattern string.
```bash
$ grep -vi udp /etc/services
```
* **Recursive Search (`-r`):** Scans all files recursively within a target directory structure.
* **Filename Isolation (`-l`):** Prints only the names of files containing the matching text pattern, rather than 
displaying the matching lines.
* **Filename Exclustion (`-L`):** Prints only the names of files not containing the matching text pattern.
```bash
$ grep -rli peerdns /usr/share/doc/
/usr/share/doc/dnsmasq-2.66/setup.html
/usr/share/doc/initscripts-9.49.17/sysconfig.txt
```
* **Color Formatting Highlights (`--color`):** Highlights matching text strings in the terminal output (defaulting to red) to improve readability.
```bash
$ grep -ri --color root /etc/systemd/
/etc/systemd/logind.conf:#KillExcludeUsers=root
```

---

### 3. Filtering Standard Output Streams via Pipes (`|`)
`grep` can filter the output of other terminal commands by accepting input via a standard Linux pipeline stream (`|`).

#### Example: Filtering Interface Configurations
Instead of reading through the entire output of the network configuration tool `ip addr show`, you can pipe the output to `grep` to display only the lines containing IP addresses (`inet`):

```bash
$ ip addr show | grep inet
inet 127.0.0.1/8 scope host lo
inet 192.168.1.231/24 brd 192.168.1.255 scope global wlan0
```