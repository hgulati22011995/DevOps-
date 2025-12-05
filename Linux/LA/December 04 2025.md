# Linux Admin Day 2: Storage, Systems & Networking

A comprehensive guide derived from Day 2 (Dec 4th) practical notes. This document covers the critical "plumbing" of Linux administration: managing disks, controlling background services, and configuring network connectivity.

## Table of Contents

- [Linux Admin Day 2: Storage, Systems \& Networking](#linux-admin-day-2-storage-systems--networking)
  - [Table of Contents](#table-of-contents)
  - [Phase 1: Advanced Storage Management](#phase-1-advanced-storage-management)
    - [Mounting \& Filesystems](#mounting--filesystems)
    - [Partitioning Tools (fdisk/gdisk/parted)](#partitioning-tools-fdiskgdiskparted)
    - [Logical Volume Management (LVM)](#logical-volume-management-lvm)
    - [Disk Usage Analysis](#disk-usage-analysis)
  - [Phase 2: System \& Process Management](#phase-2-system--process-management)
    - [Process Monitoring](#process-monitoring)
    - [Systemd \& Services](#systemd--services)
    - [System Targets \& Logs](#system-targets--logs)
    - [Automation (Cron)](#automation-cron)
  - [Phase 3: Networking Fundamentals](#phase-3-networking-fundamentals)
    - [IP Addressing \& Subnetting](#ip-addressing--subnetting)
    - [Network Configuration (nmcli/ip)](#network-configuration-nmcliip)
    - [Connectivity Testing](#connectivity-testing)
    - [DNS Configuration](#dns-configuration)
  - [Phase 4: Network Security](#phase-4-network-security)
    - [SSH \& Key-Based Auth](#ssh--key-based-auth)
    - [Firewall Management (Firewalld)](#firewall-management-firewalld)

---

## Phase 1: Advanced Storage Management
<a name="phase-1-advanced-storage-management"></a>

### Mounting & Filesystems

**Mounting** is the act of attaching a storage device to a specific directory (Mount Point) so the OS can access it.

**Commands:**
```bash
# List all block devices (Tree view)
lsblk

# Mount a partition manually
sudo mount /dev/sdb1 /mnt

# Unmount a partition
sudo umount /mnt
# OR
sudo umount /dev/sdb1

# Check if a device is busy (if umount fails)
fuser -m /mnt
```

**Permanent Mounting:**
To make mounts persist after a reboot, edit the `/etc/fstab` file.

**Mount Point Locations:**
* `/media`: Automatic mounts (USB drives, CDs).
* `/mnt`: Manual/Temporary mounts.

---

### Partitioning Tools (fdisk/gdisk/parted)

Partitioning splits a physical disk into isolated sections.

| Tool | Format Support | Best Used For |
| :--- | :--- | :--- |
| **fdisk** | **MBR** (Legacy) | Older systems, disks < 2TB. |
| **gdisk** | **GPT** (Modern) | UEFI systems, disks > 2TB. |
| **parted** | **MBR & GPT** | Scripting/Automation. |

**MBR vs GPT:**
* **MBR**: Max 4 primary partitions, Max 2TB disk size.
* **GPT**: Max 128 partitions, Huge disk sizes, requires UEFI.

**Interactive Commands (inside fdisk/gdisk):**
* `p`: Print partition table.
* `n`: New partition.
* `d`: Delete partition.
* `w`: Write changes to disk.

**Important**: After partitioning, you MUST format the partition (e.g., `mkfs.ext4 /dev/sdb1`) before mounting.

---

### Logical Volume Management (LVM)
<a name="logical-volume-management-lvm"></a>

LVM allows you to resize storage dynamically without downtime and combine multiple physical disks into one "Logical" storage pool.

**The Hierarchy:**
1.  **PV (Physical Volume)**: The raw disk (`/dev/sdb`).
2.  **VG (Volume Group)**: A pool of storage created from PVs.
3.  **LV (Logical Volume)**: The slice of the VG you actually mount and use.

**LVM Workflow:**
```bash
# 1. Initialize Physical Volume
pvcreate /dev/sda1

# 2. Create Volume Group
vgcreate vg_data /dev/sda1

# 3. Create Logical Volume (10GB size)
lvcreate -L 10G -n lv_data vg_data

# 4. Format and Mount
mkfs.ext4 /dev/vg_data/lv_data
mount /dev/vg_data/lv_data /mnt
```

```bash
# Resize Logical Volume (e.g., increase by 5GB)
lvextend -L +5G /dev/vg_data/lv_data
# -L: specify new size or +size to increase

# Resize filesystem to use new space
resize2fs /dev/vg_data/lv_data

# Reduce Logical Volume (e.g., decrease by 2GB) - DANGEROUS
# 1. First, shrink filesystem
resize2fs /dev/vg_data/lv_data 8G

# 2. Then reduce LV size
lvreduce -L 8G /dev/vg_data/lv_data

# 3. Verify
df -h /mnt
```

**Status Commands:** `pvs` (Physical), `vgs` (Groups), `lvs` (Logical).

---

### Disk Usage Analysis
* **`df` (Disk Free)**: Checks **Filesystem** capacity.
    * `df -h`: Human readable format.
* **`du` (Disk Usage)**: Checks **File/Folder** size.
    * `du -sh /home`: Summary of specific folder size.

---

## Phase 2: System & Process Management
<a name="phase-2-system--process-management"></a>

### Process Monitoring
A process is any running program.

**Tools:**
* `ps`: View current terminal processes.
* `ps aux`: Snapshot of all running processes.
* `top` / `htop`: Real-time interactive view of CPU/RAM usage.
* `kill [PID]`: Stop a process by ID.
* `killall [process_name]`: Stop all instances of a process.
* `nice` / `renice`: Adjust process priority.
* `free -h`: Check RAM usage.
* `vmstat`: System performance metrics.
* `kill -9 [PID]`: Force kill a stubborn process.
* `uptime`: System load averages.

### Systemd & Services
**Systemd** is the manager that controls how Linux starts and runs services (background programs like web servers).

**Essential Commands:**
```bash
systemctl status sshd      # Check if running
sudo systemctl start sshd  # Start service
sudo systemctl stop sshd   # Stop service
sudo systemctl restart sshd # Restart service
sudo systemctl enable sshd # Start automatically on BOOT
sudo systemctl disable sshd # Disable auto-start on BOOT
sudo systemctl list-units --type=service  # List all services
sudo systemctl is-active sshd  # Check if service is active
sudo systemctl is-enabled sshd # Check if service is enabled at boot
sudo systemctl reload sshd  # Reload config without stopping
sudo systemctl daemon-reload  # Reload systemd manager config
sudo systemctl mask sshd   # Prevent service from starting
sudo systemctl unmask sshd # Allow service to start again
```

### System Targets & Logs
**Targets** are system states (similar to Runlevels).
* `multi-user.target`: CLI only (Servers).
* `graphical.target`: GUI Desktop.

**Logging (`journalctl`)**:
Systemd's centralized logging tool.
```bash
journalctl           # View all logs
journalctl -u sshd   # View logs for SSH service only
journalctl -f        # Follow live logs (Real-time)
journalctl -xe       # View recent error logs
# xe : e for extended, shows more details about errors.
journalctl --since "2025-12-01" --until "2025-12-04 12:00"  # Logs between specific dates/times
journalctl -p err..alert  # View logs with priority from error to alert
# p : priority level filter

journalctl --disk-usage  # Check journal log size
journalctl --vacuum-size=100M  # Reduce log size to 100MB
```
*Note: `/var/log` stores traditional plain-text log files (syslog, auth.log).*

### Automation (Cron)
Schedule tasks to run automatically at specific times.

* `crontab -e`: Edit user's scheduled jobs.
* `crontab -l`: List jobs.
* **Format**: `Minute Hour Day Month Weekday Command`
    * Example: `30 2 * * * /backup.sh` (Runs daily at 2:30 AM).

---

## Phase 3: Networking Fundamentals
<a name="phase-3-networking-fundamentals"></a>

### IP Addressing & Subnetting
* **IP Address**: Unique device ID (e.g., `192.168.1.10`).
* **Gateway**: The router that connects you to the internet (`192.168.1.1`).
* **Subnet Mask / CIDR**: Defines network size (`/24` = 255.255.255.0).

### Network Configuration (nmcli/ip)
Avoid `ifconfig` (deprecated). Use modern tools:

**1. `ip` command (Low-level status)**:
* `ip a`: Show IP addresses.
* `ip r`: Show routing table (gateway).

**2. `nmcli` (Network Manager CLI - Configuration)**:
```bash
# Check status
nmcli connection show

# Edit Connection (Set static IP)
nmcli connection modify eth0 ipv4.addresses 192.168.1.10/24
nmcli connection modify eth0 ipv4.gateway 192.168.1.1
nmcli connection modify eth0 ipv4.method manual

# Apply Changes
nmcli connection down eth0 && nmcli connection up eth0
```

### Connectivity Testing
* `ping google.com`: Check reachability/latency.
* `curl http://site.com`: Test Web/API access.
* `ss -tulnp`: Check open ports (replaces `netstat`).
* `telnet [ip] [port]`: Test if a specific port is open.

### DNS Configuration
DNS translates names (`google.com`) to IPs.
* Config file: `/etc/resolv.conf`
* Test tools: `nslookup google.com` or `dig google.com`.

---

## Phase 4: Network Security
<a name="phase-4-network-security"></a>

### SSH & Key-Based Auth
Secure remote login.

**Workflow for Password-less Login:**
1.  **Generate Key Pair** (On your machine):
    `ssh-keygen` (Creates private `id_rsa` and public `id_rsa.pub`).
2.  **Copy Public Key to Server**:
    `ssh-copy-id user@server_ip`.
3.  **Login**:
    `ssh user@server_ip` (No password needed).

### Firewall Management (Firewalld)
Controls allowed network traffic using "Zones".

**Commands:**
```bash
# Check status
systemctl status firewalld

# List active rules
sudo firewall-cmd --list-all

# Open a port (e.g., Web Server) PERMANENTLY
sudo firewall-cmd --add-port=80/tcp --permanent

# Close a port
sudo firewall-cmd --remove-port=80/tcp --permanent

# Reload to apply changes
sudo firewall-cmd --reload
```