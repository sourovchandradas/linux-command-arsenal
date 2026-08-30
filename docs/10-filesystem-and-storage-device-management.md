# Filesystem and Storage Device Management

## Table of Contents

- [Introduction](#introduction)
- [Storage Device Naming and Partitions](#storage-device-naming-and-partitions)
- [Character vs. Block Devices](#character-vs-block-devices)
- [Inspecting Storage: fdisk vs. lsblk](#inspecting-storage-fdisk-vs-lsblk)
- [Mounting and Unmounting Filesystems](#mounting-and-unmounting-filesystems)
- [Monitoring and Repairing Filesystems (df and fsck)](#monitoring-and-repairing-filesystems-df-and-fsck)
- [Key Tips](#key-tips)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

---

## Introduction

Unlike Windows (which uses physical drive letters like `C:`, `D:`, or `E:`), Linux represents all attached hardware and storage devices as individual files under the `/dev` directory, mounted onto a unified single root file tree (`/`). For security professionals and system administrators, understanding device management is critical for navigating target environments, transferring tools via external media, identifying confidential data storage locations, and managing physical drives securely.

---

## Storage Device Naming and Partitions

Linux assigns logical device labels based on drive hardware types and serial attachment order.

### Device Naming Evolution

| Drive Type | Device File Format | Example | Description |
| --- | --- | --- | --- |
| **Floppy Drive** | `fd<number>` | `fd0` | Legacy floppy disk drive |
| **IDE / E-IDE Hard Drive** | `hd<letter>` | `hda`, `hdb` | Legacy parallel ATA hard drives |
| **SATA / SCSI / USB Drive** | `sd<letter>` | `sda`, `sdb`, `sdc` | Modern serial storage devices (alphabetical by drive) |
| **Optical Drive** | `sr<number>` | `sr0` | CD/DVD-ROM drive |

### Partition Labeling Scheme

Partitions divide a single physical drive into logical sections. Linux represents partitions by appending a minor number after the major drive letter.

| Device File | Component | Description |
| --- | --- | --- |
| `sda1` | `sd` (SATA) + `a` (Drive 1) + `1` (Partition 1) | First partition on the first SATA hard drive |
| `sda2` | `sd` (SATA) + `a` (Drive 1) + `2` (Partition 2) | Second partition on the first SATA hard drive |
| `sda5` | `sd` (SATA) + `a` (Drive 1) + `5` (Partition 5) | Extended/Swap partition (Virtual RAM overflow) |
| `sdb1` | `sd` (SATA) + `b` (Drive 2) + `1` (Partition 1) | First partition on the second storage device (e.g., USB drive) |

### Native vs. Non-Native Filesystems

| Filesystem Type | System Origin | Partition ID Code (fdisk) | Notes |
| --- | --- | --- | --- |
| **ext2 / ext3 / ext4** | Linux | `83` (Linux) | Standard native Linux filesystems (`ext4` is latest) |
| **Linux Swap** | Linux | `82` (Linux swap) | Dedicated partition used for virtual RAM overflow |
| **NTFS / HPFS / exFAT** | Windows / macOS | `7` (HPFS/NTFS/exFAT) | Indicates media formatted on non-Linux OS |
| **FAT / FAT32** | Legacy Windows | `b` / `c` | Common compatibility format for USB flash drives |

---

## Character vs. Block Devices

Device files inside `/dev` transfer data in one of two modes, indicated by the first letter in `ls -l` permission strings.

| Device Type | Mode Flag | Data Transfer Method | Speed / Throughput | Hardware Examples |
| --- | --- | --- | --- | --- |
| **Character Device** | `c` | Character-by-character (stream) | Lower speed | Keyboards, mice, terminal serial ports |
| **Block Device** | `b` | Data blocks (multiple bytes per chunk) | High speed | Hard drives (HDD/SSD), USB flash drives, DVD drives |

```bash
# Long listing of /dev showing character (c) and block (b) flags
crw------- 1 root root  10, 175 May 16 12:44 agpgart    # Character device
brw-rw---- 1 root root   8,   0 May 16 12:44 sda        # Block device (Hard Drive)

```

---

## Inspecting Storage: fdisk vs. lsblk

Linux provides multiple utilities to list physical drives, partition tables, and filesystem specifications.

### Inspection Utility Comparison

| Feature | `fdisk -l` | `lsblk` |
| --- | --- | --- |
| **Root Privileges** | **Required** (`sudo` / `root`) | **Not Required** (Runs under standard user) |
| **Output Style** | Detailed disk geometry, sector counts, Partition IDs | Visual tree-view of block devices & mount points |
| **Primary Use** | Partition table inspection and partition editing | Quick overview of disk topology and active mount points |

### Execution Examples

```bash
# 1. Detailed partition inspection (Requires root)
fdisk -l

# Example fdisk output snippet:
# Device     Boot     Start       End   Sectors    Size  Id Type
# /dev/sda1    *       2048  39174143  39172096   18.7G  83 Linux
# /dev/sda5        39176192  41940991   2764800    1.3G  82 Linux swap / Solaris
# /dev/sdb1              32  62498815  62498784   29.8G   7 HPFS/NTFS/exFAT

# 2. Tree-view block layout (No root required)
lsblk

# Example lsblk output snippet:
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0   20G  0 disk 
# |-sda1   8:1    0 18.7G  0 part /
# |-sda5   8:5    0  1.3G  0 part [SWAP]
# sdb      8:16   1 29.8G  0 disk 
# |-sdb1   8:17   1 29.8G  0 part /media

```

---

## Mounting and Unmounting Filesystems

Connecting a storage device physically to a system does not automatically make its files accessible. Devices must be logically attached to an empty directory in the root tree via **mounting**.

### Mount Directory Conventions

* `/media`: Standard directory used by Linux distributions for **automatically mounted** removable drives.
* `/mnt`: Standard directory convention for **manually mounted** storage devices.
* `/etc/fstab`: Filesystem Table file read at boot time to mount permanent drives automatically.

### Mounting Workflow & Rules

> **Warning:** Always mount devices to an **empty directory**. Mounting over a directory containing existing files hides those original files until the device is unmounted.

```bash
# Mount a partition manually to /mnt
mount /dev/sdb1 /mnt

# Mount a flash drive to /media
mount /dev/sdc1 /media

```

### Unmounting with umount

Before physically removing a storage device, it must be logically detached to prevent data corruption.

```bash
# Correct command spelling is 'umount' (NO 'n')
umount /dev/sdb1

```

> **Error Handling:** If a process is actively reading or writing to the target device, `umount` will fail with a `target is busy` error. Close active terminal sessions or files operating inside the mount point before unmounting.

---

## Monitoring and Repairing Filesystems (df and fsck)

### Monitoring Disk Space with df

The `df` (disk free) command displays total capacity, used space, available space, and active mount points across all mounted storage devices (measured in 1KB blocks by default).

```bash
# Display space for all mounted filesystems
df

# Display space for a specific drive
df /dev/sdb1

```

### Checking and Repairing Filesystems with fsck

The `fsck` (filesystem check) utility inspects drives for sector errors, repairs damaged structures, or logs unrecoverable bad blocks.

> **CRITICAL RULE:** **NEVER** run `fsck` on a mounted filesystem. Running `fsck` on an active drive can cause severe data corruption. Always unmount first.

```bash
# Step 1: Unmount the target drive
umount /dev/sdb1

# Step 2: Run fsck with automatic repair flag (-p)
fsck -p /dev/sdb1

```

---

## Key Tips

* **`umount` Command Spelling** — The command is spelled `umount` (without an `n` after the `u`).
* **Always Unmount Before `fsck**` — Attempting to repair a mounted drive will cause `e2fsck` to abort to prevent filesystem destruction.
* **Directory Overwrite Caution** — Mounting to a non-empty directory hides existing content temporarily; unmounting restores visibility.
* **Root Privilege Differences** — Use `lsblk` for user-level storage layout checks when root access is unavailable for `fdisk -l`.
* **Identifying Host Origins** — Spotting `NTFS` or `exFAT` partition types during analysis indicates media formatted on Windows/macOS.

---

## Quick Command Chains

```bash
# Safe inspection, unmount, repair, and remount cycle
lsblk && umount /dev/sdb1 && fsck -p /dev/sdb1 && mount /dev/sdb1 /mnt

# Check disk usage human-readable (-h) on active USB mount
df -h /media/*

```

---

## Quick Start

Inspect Block Layout (`lsblk`) $\rightarrow$ Mount Drive (`mount /dev/sdb1 /mnt`) $\rightarrow$ Monitor Usage (`df`) $\rightarrow$ Safely Detach (`umount /dev/sdb1`) $\rightarrow$ Repair Errors (`fsck -p /dev/sdb1`)
