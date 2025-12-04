# December 03 2025

A comprehensive mastery guide derived from practical lab notes (Dec 3rd) and expanded with industry-standard Linux Administrator knowledge. This guide is structured to take you from understanding the core kernel to managing complex systems.

## Table of Contents

- [Decem](#decem)
  - [Table of Contents](#table-of-contents)
  - [Phase 1: Linux Fundamentals \& Architecture](#phase-1-linux-fundamentals--architecture)
    - [The Operating System \& Kernel](#the-operating-system--kernel)
    - [The Boot Process (GRUB)](#the-boot-process-grub)
    - [The File System Hierarchy](#the-file-system-hierarchy)
  - [Phase 2: The Shell Environment](#phase-2-the-shell-environment)
    - [Understanding the Shell](#understanding-the-shell)
    - [Environment Variables \& PATH](#environment-variables--path)
    - [Configuration: .bashrc](#configuration-bashrc)
  - [Phase 3: Package Management](#phase-3-package-management)
    - [Package Managers (APT/DNF)](#package-managers-aptdnf)
  - [Phase 4: User \& Security Management](#phase-4-user--security-management)
    - [User Lifecycle (Add/Mod/Del)](#user-lifecycle-addmoddel)
    - [Group Management](#group-management)
    - [Critical Files: passwd \& shadow](#critical-files-passwd--shadow)
    - [Sudoers \& Privileges](#sudoers--privileges)
    - [File Permissions (chmod/chown)](#file-permissions-chmodchown)
  - [Phase 5: Disk \& Storage Administration](#phase-5-disk--storage-administration)
    - [Disk Usage (df vs du)](#disk-usage-df-vs-du)
    - [Mounting \& Partitions (lsblk/mount)](#mounting--partitions-lsblkmount)
  - [Phase 6: Networking \& Data Transfer](#phase-6-networking--data-transfer)
    - [The curl Command](#the-curl-command)
    - [wget](#wget)
    - [rsync (Synchronization)](#rsync-synchronization)
  - [Phase 7: Advanced Shell Operations](#phase-7-advanced-shell-operations)
    - [Redirection \& Pipes](#redirection--pipes)
    - [Text Processing (sed)](#text-processing-sed)
    - [Archiving (tar, zip)](#archiving-tar-zip)
  - [Phase 8: Text Editing (Vim)](#phase-8-text-editing-vim)
  - [Bonus: ITIL Basics](#bonus-itil-basics)

---

## Phase 1: Linux Fundamentals & Architecture
<a name="phase-1-linux-fundamentals--architecture"></a>

### The Operating System & Kernel

**What is an OS?**
An Operating System (OS) is system software that acts as a bridge between the user and the computer hardware. It manages resources like CPU, Memory (RAM), and Storage.

**What is Linux?**
Linux is technically **just the Kernel**. When we say "Linux OS", we are referring to a **Distribution (Distro)** which combines:
1.  **The Linux Kernel** (The core)
2.  **System Tools** (Compilers, libraries)
3.  **Applications** (Shell, GUI, Server software)

**The Kernel**
The kernel is a program written in low-level languages (like C) that has direct access to hardware.
* **Role**: Manages processes, memory allocation, device I/O, and the file system.
* **Why it's needed**: Users cannot talk to hardware directly; it is unsafe. The kernel acts as the secure intermediary.

---

### The Boot Process (GRUB)
<a name="the-boot-process-grub"></a>

How does Linux start?
1.  **Power ON**: BIOS/UEFI runs hardware checks (POST).
2.  **Bootloader Loads**: The BIOS loads the Bootloader (usually **GRUB**).
3.  **Kernel Loads**: GRUB loads the Linux Kernel into memory.
4.  **Init System**: The Kernel starts the first process (Systemd or Init) which starts all other services.

**What is GRUB?**
* **GRand Unified Bootloader**.
* It allows you to choose which OS to boot (if dual booting) and handles boot parameters.

---

### The File System Hierarchy
<a name="the-file-system-hierarchy"></a>

Linux follows a strict directory structure. Here is what every Admin must know:

| Directory | Purpose |
| :--- | :--- |
| **/** | The Root directory. The top of the tree. |
| **/root** | Home directory for the **root** (admin) user. (Not the same as `/`). |
| **/home** | Home directories for normal users (e.g., `/home/john`). |
| **/bin** | **Essential** User Binaries (commands like `ls`, `cp`, `cat`). |
| **/sbin** | **System** Binaries. Admin-only commands (e.g., `reboot`, `iptables`, `fdisk`). |
| **/etc** | **Configuration** files. (e.g., `/etc/passwd`, `/etc/ssh/sshd_config`). |
| **/var** | **Variable** data. Files that grow/change (Logs `/var/log`, Web files `/var/www`). |
| **/tmp** | **Temporary** files. Often cleared on reboot. |
| **/dev** | **Device** files. Represents hardware (e.g., `/dev/sda` is a hard drive). |
| **/proc** | **Process** information. A virtual filesystem reflecting kernel state. |
| **/opt** | **Optional** add-on software (Google Chrome, proprietary apps). |
| **/mnt** | **Mount** point for temporary filesystems (USB sticks, test drives). |
| **/media** | Automount point for removable media (CD-ROMs). |
| **/lib** | Shared libraries needed by binaries in `/bin` and `/sbin`. |

---

## Phase 2: The Shell Environment
<a name="phase-2-the-shell-environment"></a>

### Understanding the Shell
The shell is the user interface to the kernel. You type a command, the shell interprets it, and tells the kernel what to do.
* **Common Shells**: Bash (Standard), Zsh (Modern features), Sh (Legacy).

### Environment Variables & PATH
<a name="environment-variables--path"></a>

**What is the `PATH`?**
The `PATH` variable tells the shell **where to look** for executable programs. It is a list of directories separated by colons (`:`).

**How it works:**
When you type `ls`:
1.  Linux looks in `/usr/local/bin`. Is `ls` there? No.
2.  Linux looks in `/usr/bin`. Is `ls` there? Yes.
3.  Linux runs `/usr/bin/ls`.

**Commands:**
```bash
# View current PATH
echo $PATH

# Temporarily add a folder to PATH (Lost after closing terminal)
export PATH=$PATH:/opt/myapp/bin
```

### Configuration: .bashrc
<a name="configuration-bashrc"></a>

* **File**: `~/.bashrc` (Located in user's home directory).
* **Purpose**: A script that runs **every time** you open a new terminal window.
* **Use Case**: Storing permanent PATH changes, aliases, and custom prompts.

**How to make changes permanent:**
1.  Open file: `nano ~/.bashrc`
2.  Add line: `export PATH=$PATH:/opt/myapp/bin`
3.  Save and Exit.
4.  **Apply changes**: `source ~/.bashrc`

---

## Phase 3: Package Management
<a name="phase-3-package-management"></a>

Software in Linux is managed via repositories to ensure security and dependency handling.

### Package Managers (APT/DNF)

**Debian/Ubuntu uses `apt`**:
```bash
sudo apt update          # Refresh repository list
sudo apt upgrade         # Update all installed software
sudo apt install nginx   # Install a package
sudo apt remove nginx    # Remove a package
```

**RHEL/CentOS/Rocky uses `dnf` (or `yum`):**
```bash
sudo dnf check-update    # Check for updates
sudo dnf update          # Update system
sudo dnf install httpd   # Install a package
sudo dnf remove httpd    # Remove a package
```

**Repositories**: Secure servers where the package manager downloads software from.

---

## Phase 4: User & Security Management
<a name="phase-4-user--security-management"></a>

### User Lifecycle (Add/Mod/Del)

**Creating Users:**
```bash
# Create user 'john' AND create his home directory (-m)
sudo useradd -m john

# Set a password for john (useradd does not do this automatically)
sudo passwd john
```

**Modifying Users (`usermod`):**
```bash
# Add john to the 'docker' group (-a = append, -G = group)
sudo usermod -aG docker john

# Change john's default shell to bash
sudo usermod -s /bin/bash john
```

**Deleting Users (`userdel`):**
```bash
# Delete user 'john' AND his home directory (-r)
sudo userdel -r john
```

### Group Management
* **Primary Group**: The default group files belong to when you create them (usually same as username).
* **Secondary Group**: Extra groups for permissions (e.g., `sudo`, `docker`, `admin`).

```bash
sudo groupadd developers       # Create a group
sudo usermod -aG developers ali # Add user to group
groups ali                     # View user's groups
```

### Critical Files: passwd & shadow
<a name="critical-files-passwd--shadow"></a>

1.  **`/etc/passwd`**:
    * Stores user account details (UID, GID, Home Dir, Shell).
    * **Readable by everyone**.
    * Format: `user:x:1000:1000:comment:/home/user:/bin/bash`
2.  **`/etc/shadow`**:
    * Stores **Encrypted Passwords** and expiration rules.
    * **Readable only by root**.
    * This separation exists for security.

### Sudoers & Privileges
**`visudo`**: The command used to safely edit the `/etc/sudoers` file. It prevents syntax errors that could lock you out of the system.

**NOPASSWD Rule**:
To allow a user to run a specific command without typing a password:
```text
# Inside sudoers file
john ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```

### File Permissions (chmod/chown)
*(Expanded concept)*

**Ownership (`chown`)**:
```bash
# Change owner of file to 'john'
sudo chown john file.txt
# Change owner to 'john' and group to 'devs' recursively
sudo chown -R john:devs /var/www/html
```

**Permissions (`chmod`)**:
Read (r=4), Write (w=2), Execute (x=1).

```bash
# Give User (u) Read/Write/Execute (7), Group (g) Read/Execute (5), Others (o) Read (4)
chmod 754 script.sh
# Make a file executable
chmod +x script.sh
```

---

## Phase 5: Disk & Storage Administration
<a name="phase-5-disk--storage-administration"></a>

### Disk Usage (df vs du)

**`df` (Disk Free)**:
Checks the **filesystem** level. Shows total block availability.
```bash
df -h          # Human readable sizes (GB, MB)
df -h /home    # Check specific partition
df -i          # Check Inode usage (file count limit)
```

**`du` (Disk Usage)**:
Checks actual **file/directory** sizes.
```bash
du -sh /var/log    # Summary (-s) Human-readable (-h) size of folder
du -h --max-depth=1 /var  # Find which subfolder is eating space
```

**Why they differ**: If you delete a large log file but a process is still writing to it, `df` won't show the space as free until the process is killed. `du` will show it as gone.

### Mounting & Partitions (lsblk/mount)

**`lsblk`**: Lists all block devices (disks/partitions) in a tree view.
```bash
lsblk -f       # Also shows filesystem types (ext4, xfs)
```

**`mount` / `umount`**:
Attaching a storage device to a folder (Mount Point).
```bash
# Mount /dev/sdb1 to /mnt
sudo mount /dev/sdb1 /mnt

# Unmount
sudo umount /mnt
```

**`/etc/fstab`**: The file used to make mounts permanent across reboots.

---

## Phase 6: Networking & Data Transfer
<a name="phase-6-networking--data-transfer"></a>

### The curl Command
Versatile tool for transferring data via URL.

```bash
# Download and save with original name
curl -O [https://example.com/file.zip](https://example.com/file.zip)

# Download and rename
curl -o newname.zip [https://example.com/file.zip](https://example.com/file.zip)

# Test an API (GET)
curl [https://api.example.com/users](https://api.example.com/users)

# Send Data (POST)
curl -X POST -d "user=john" [https://api.site.com](https://api.site.com)

# Check Headers only (Debug server issues)
curl -I [https://google.com](https://google.com)
```

### wget
Simpler tool, primarily for downloading files.
```bash
wget [https://example.com/file.iso](https://example.com/file.iso)
```

### rsync (Synchronization)
The gold standard for backups. It only copies **changed** parts of files.

```bash
# Local Copy (Archive mode -a, Verbose -v)
rsync -av /source/ /destination/

# Remote Copy (Push to server)
rsync -av /local/data/ user@192.168.1.50:/remote/backup/
```

---

## Phase 7: Advanced Shell Operations
<a name="phase-7-advanced-shell-operations"></a>

### Redirection & Pipes

| Symbol | Name | Function | Example |
| :--- | :--- | :--- | :--- |
| `>` | Output Redirection | Sends output to file (Overwrites). | `ls > file.txt` |
| `>>` | Append Redirection | Adds output to end of file. | `date >> log.txt` |
| `<` | Input Redirection | Feeds file content to command. | `wc -l < file.txt` |
| `2>` | Error Redirection | Sends error messages to file. | `ls wrongfile 2> error.log` |
| `|` | Pipe | Sends output of Cmd1 to Input of Cmd2. | `cat file.txt | grep "Error"` |

### Text Processing (sed)
**Stream Editor**: Used for programmatic text editing.

```bash
# Replace 'old' with 'new' (Output only)
sed 's/old/new/' file.txt

# Replace ALL occurrences globally (g)
sed 's/old/new/g' file.txt

# Delete line 3
sed '3d' file.txt

# EDIT THE FILE IN-PLACE (-i) - Dangerous, use with care!
sed -i 's/error/fixed/g' log.txt
```

### Archiving (tar, zip)

**`tar` (Tape Archive)**:
* **Create**: `tar -cvf archive.tar files/`
* **Extract**: `tar -xvf archive.tar`
* **Compress (gzip)**: `tar -czvf archive.tar.gz files/`
* **Extract (gzip)**: `tar -xzvf archive.tar.gz`

**`zip`**:
* **Zip**: `zip -r data.zip folder/`
* **Unzip**: `unzip data.zip -d /target/folder`

---

## Phase 8: Text Editing (Vim)
<a name="phase-8-text-editing-vim"></a>

Vim is a modal editor. You are always in one of 3 modes:

1.  **Command Mode (Default)**:
    * `dd`: Delete (Cut) line.
    * `yy`: Yank (Copy) line.
    * `p`: Paste.
    * `u`: Undo.
    * `x`: Delete character.

2.  **Insert Mode (Typing)**:
    * Press `i` to start typing.
    * Press `Esc` to go back to Command Mode.

3.  **Last Line Mode (Saving)**:
    * Press `:` to enter.
    * `:w`: Save.
    * `:q`: Quit.
    * `:wq`: Save and Quit.
    * `:q!`: Force Quit (No save).

---

## Bonus: ITIL Basics
<a name="bonus-itil-basics"></a>

**ITIL (Information Technology Infrastructure Library)** is a framework of best practices for delivering IT services.

* **Goal**: Align IT services with business needs.
* **Key Benefits**: Consistency, Efficiency, Customer Satisfaction.
* **ITSM**: IT Service Management—the actual practice of implementing ITIL.
* **Versions**: ITIL v3 (Process-oriented) vs ITIL 4 (Value-stream oriented, more agile).
   