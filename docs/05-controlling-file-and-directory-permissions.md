# Controlling File and Directory Permissions

## Table of Contents

- [Introduction](#introduction)
- [Permission Types and File Structure](#permission-types-and-file-structure)
- [Granting Ownership (chown and chgrp)](#granting-ownership-chown-and-chgrp)
- [Modifying Permissions with chmod](#modifying-permissions-with-chmod)
- [Default Permissions and umask](#default-permissions-and-umask)
- [Special Permissions (SUID, SGID, Sticky Bit)](#special-permissions-suid-sgid-sticky-bit)
- [Privilege Escalation and Audit (Hacker Perspective)](#privilege-escalation-and-audit-hacker-perspective)
- [Key Tips](#key-tips)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

---

## Introduction

In a Linux multi-user system, file and directory permissions enforce access control and system security. The system administrator (`root`) or the file owner defines permission status for three identity tiers: the file **Owner** (`u`), the assigned **Group** (`g`), and **Others** (`o`). Understanding these controls is essential for configuring security boundaries, executing scripts, and auditing system vulnerabilities.

---

## Permission Types and File Structure

### Access Levels

| Permission | Symbol | Binary | Octal Value | Meaning on File | Meaning on Directory |
| --- | --- | --- | --- | --- | --- |
| **Read** | `r` | `100` | `4` | View file contents | List directory contents (`ls`) |
| **Write** | `w` | `010` | `2` | Modify/edit file contents | Create or delete files inside directory |
| **Execute** | `x` | `001` | `1` | Run file as a program/script | Enter directory (`cd`) and access items |

### Inspecting Permissions (`ls -l`)

```bash
# Long listing to inspect permissions, ownership, size, and timestamp
ls -l /usr/share/hashcat

# Example string analysis: -rwxr-xr-- 1 root root 33685504 June 28 hashcat.hcstat
# Field Breakdown:
# [-]           : File type (- = regular file, d = directory)
# [rwx]         : Owner permissions (Read, Write, Execute = 7)
# [r-x]         : Group permissions (Read, Execute = 5)
# [r--]         : Others permissions (Read only = 4)
# [1]           : Number of hard links
# [root root]   : File Owner username, File Group name

```

---

## Granting Ownership (chown and chgrp)

| Command | Purpose | Syntax Example |
| --- | --- | --- |
| `chown` | Reassign individual user ownership | `chown bob /tmp/bobsfile` |
| `chgrp` | Reassign group ownership | `chgrp security newIDS` |
| `chown` (Both) | Reassign user and group simultaneously | `chown root:security /opt/tool` |

```bash
# Transfer file ownership to user 'bob'
chown bob /tmp/bobsfile

# Reassign group access of tool 'newIDS' to team group 'security'
chgrp security newIDS

```

---

## Modifying Permissions with chmod

### Method 1: Octal / Decimal Notation

Octal representation sums the binary values of active permissions ($r=4, w=2, x=1$) for each identity class:

| Octal Digit | Permission String | Binary | Description |
| --- | --- | --- | --- |
| **`0`** | `---` | `000` | No permissions granted |
| **`1`** | `--x` | `001` | Execute only |
| **`2`** | `-w-` | `010` | Write only |
| **`3`** | `-wx` | `011` | Write and Execute ($2+1$) |
| **`4`** | `r--` | `100` | Read only |
| **`5`** | `r-x` | `101` | Read and Execute ($4+1$) |
| **`6`** | `rw-` | `110` | Read and Write ($4+2$) |
| **`7`** | `rwx` | `111` | Read, Write, and Execute ($4+2+1$) |

```bash
# Grant Owner all (7), Group all (7), Others read-only (4)
chmod 774 hashcat.hcstat

# Grant Owner read/write/execute (7), Group and Others read/write (6)
chmod 766 newhackertool

```

### Method 2: Symbolic (UGO) Notation

Uses targets (`u`ser, `g`roup, `o`thers, `a`ll), operators (`+` add, `-` remove, `=` set), and permissions (`r`, `w`, `x`).

```bash
# Remove write permission from user (owner)
chmod u-w hashcat.hcstat

# Add execute permission to user and others simultaneously
chmod u+x,o+x hashcat.hcstat

```

---

## Default Permissions and umask

Linux assigns base permissions of **`666`** (files) and **`777`** (directories) upon creation. The `umask` value subtracts bits from base permissions to enforce security.

$$\text{File Permissions} = 666 - \text{umask}$$

$$\text{Directory Permissions} = 777 - \text{umask}$$

| Base Type | Base Value | umask | Resulting Octal | Resulting String |
| --- | --- | --- | --- | --- |
| **New File** | `666` | `022` | `644` | `rw-r--r--` |
| **New Directory** | `777` | `022` | `755` | `rwxr-xr-x` |

```bash
# View current user umask setting
umask

# Set user-level custom umask in ~/.profile (Restricts group/others)
echo "umask 007" >> ~/.profile

```

---

## Special Permissions (SUID, SGID, Sticky Bit)

Special permissions precede standard permission triplets in octal notation (4-digit format).

| Special Bit | Octal Prefix | Symbol in `ls -l` | Technical Behavior | Security Context |
| --- | --- | --- | --- | --- |
| **SUID** (Set User ID) | `4000` | `s` (Owner) e.g., `-rwsr-xr-x` | Executes file with the privileges of the file **owner** | Critical privilege escalation vector |
| **SGID** (Set Group ID) | `2000` | `s` (Group) e.g., `-rwxr-sr-x` | Executes with **group** privileges; new dir files inherit group | Used for shared directory collaboration |
| **Sticky Bit** | `1000` | `t` (Others) e.g., `rwxrwxrwt` | Prevents non-owners from deleting directory files | Legacy/modern default on `/tmp` |

```bash
# Apply SUID bit (Owner executes as root)
chmod 4644 /usr/bin/custom_tool

# Apply SGID bit (Group execution / inherited group directory)
chmod 2644 /shared/project

# Set Sticky Bit on shared directory
chmod 1777 /tmp/shared_dir

```

---

## Privilege Escalation and Audit (Hacker Perspective)

Binary applications configured with SUID root (e.g., `/usr/bin/passwd` or misconfigured custom binaries) execute under `root` privileges regardless of the calling user. Attackers search for SUID binaries to execute unauthorized actions or read restricted files like `/etc/shadow`.

```bash
# Search system-wide for root-owned files with SUID bit set
find / -user root -perm -4000 2>/dev/null

# Inspect permissions of discovered SUID binary
ls -l /usr/bin/sudo

```

---

## Key Tips

* **Downloaded Tool Permissions** — Freshly downloaded scripts/binaries usually default to `666` or `644` (no execute permissions). Run `chmod 755 <tool>` or `chmod +x <tool>` before running.
* **Capital vs Lowercase (`S`/`s` and `T`/`t`)** — Lowercase `s` or `t` indicates the special bit AND execute bit are set. Uppercase `S` or `T` indicates special bit is set WITHOUT underlying execute permissions.
* **Directory Execute Bit** — Execute permission (`x`) on a directory controls whether a user can traverse (`cd`) into it, separate from read (`r`) access listing.
* **SUID Risk Assessment** — Any binary with SUID root privileges (`chmod 4000`) acts as a potential bridge for standard users to escalate to full `root` control.

---

## Quick Command Chains

```bash
# Transfer user/group ownership and set executable privileges in one pass
chown root:security script.sh && chmod 755 script.sh

# Audit all SUID binaries on the filesystem and list detailed permissions
find / -type f -perm -4000 -exec ls -ld {} \; 2>/dev/null

```

---

## Quick Start

Check permissions (`ls -l`) $\rightarrow$ Adjust ownership (`chown`/`chgrp`) $\rightarrow$ Set access modes (`chmod 755` or `u+x`) $\rightarrow$ Configure default mask (`umask`) $\rightarrow$ Audit SUID risks (`find / -perm -4000`)
