# Getting Started with Linux Basics

# Table of Contents
- [Introduction](#introduction)
- [Navigation Commands](#navigation-commands)
- [File & Directory Listing](#file--directory-listing)
- [File & Directory Operations](#file--directory-operations)
- [User & System Info](#user--system-info)
- [Finding & Searching](#finding--searching)
- [Find Command Options](#find-command-options)
- [Wildcards](#wildcards)
- [Getting Help](#getting-help)
- [Man page navigation](#man-page-navigation)
- [Important Directories](#important-directories)
- [Piping](#piping)
- [Key Tips](#key-tips)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

## Introduction

Quick reference guide for essential Linux commands : Getting Started with the Basics

---

## Navigation Commands

| Command | Usage | Example |
|---------|-------|---------|
| `pwd` | Print Working Directory - show current location | `pwd` → `/root` |
| `cd <directory>` | Change Directory | `cd /etc` |
| `cd ..` | Go up one level | `cd ..` |
| `cd /` | Go to root filesystem | `cd /` |
| `cd ~` | Go to home directory | `cd ~` |

---

## File & Directory Listing

| Command | Usage | Example |
|---------|-------|---------|
| `ls` | List directory contents | `ls` |
| `ls -l` | Long listing (detailed info) | `ls -l` |
| `ls -la` | List with hidden files | `ls -la` |
| `ls <directory>` | List specific directory | `ls /etc` |

---

## File & Directory Operations

| Command | Usage | Example |
|---------|-------|---------|
| `touch <filename>` | Create empty file | `touch newfile` |
| `cat > <filename>` | Create file (interactive) | `cat > hackingskills` |
| `cat >> <filename>` | Append content to file | `cat >> hackingskills` |
| `cat <filename>` | Display file contents | `cat hackingskills` |
| `mkdir <dirname>` | Make directory | `mkdir newdirectory` |
| `cp <source> <dest>` | Copy file | `cp oldfile /path/newfile` |
| `mv <old> <new>` | Move or rename file/directory | `mv newfile newfile2` |
| `rm <filename>` | Remove file | `rm newfile2` |
| `rmdir <dirname>` | Remove empty directory | `rmdir newdirectory` |
| `rm -r <dirname>` | Remove directory with contents | `rm -r newdirectory` |

---

## User & System Info

| Command | Usage | Example |
|---------|-------|---------|
| `whoami` | Show current user | `whoami` → `root` |
| `passwd` | Change password | `passwd` |

---

## Finding & Searching

| Command | Usage | Example |
|---------|-------|---------|
| `locate <keyword>` | Search entire filesystem | `locate aircrack-ng` |
| `whereis <binary>` | Find binary, source, man page | `whereis aircrack-ng` |
| `which <binary>` | Find binary in PATH | `which aircrack-ng` |
| `find <dir> <options>` | Powerful flexible search | `find /etc -type f -name apache2` |
| `grep <keyword>` | Filter/search for keywords | `ps aux \| grep apache2` |

---

## Find Command Options

```bash
find directory options expression

# Search by type
find / -type f -name filename      # Search for files
find / -type d -name dirname       # Search for directories

# Using wildcards
find /etc -type f -name apache2.*  # Match extensions
find /etc -type f -name *apache*   # Match anywhere
```

### Wildcards

| Wildcard | Matches | Example |
|----------|---------|---------|
| `*` | Any character(s) of any length | `*at` matches cat, hat, what, bat |
| `?` | Single character | `?at` matches cat, hat, bat (not what) |
| `[]` | Characters inside brackets | `[cb]at` matches cat, bat (not hat) |

---

## Getting Help

| Command | Usage | Example |
|---------|-------|---------|
| `<command> --help` | Show help (long form) | `aircrack-ng --help` |
| `<command> -h` | Show help (short form) | `nmap -h` |
| `man <command>` | Open manual page | `man aircrack-ng` |

## Man page navigation
- `Enter` - Scroll down
- `Pg Dn` / `Pg Up` - Page down/up
- `Arrow keys` - Move cursor
- `q` - Quit manual

---

## Important Directories

| Directory | Purpose |
|-----------|---------|
| `/` | Root of filesystem |
| `/root` | Root user's home directory |
| `/home` | User home directories |
| `/etc` | System configuration files |
| `/bin` | Essential binaries/executables |
| `/usr/bin` | More binaries |
| `/sbin` | System binaries |
| `/lib` | Libraries (like Windows DLLs) |
| `/mnt` | Mount point for filesystems |
| `/media` | Mount point for USB/CDs |
| `/dev` | Device files |
| `/proc` | Kernel data view |
| `/sys` | Kernel's hardware view |

---

## Piping

Pipe output from one command to another:

```bash
ps aux | grep apache2      # Search process list for apache2
ps aux | grep ssh          # Find SSH processes
command1 | command2        # Send output of command1 to command2
```

---

## Key Tips

✓ **Case Sensitive** - `Desktop` ≠ `desktop` ≠ `DeskTop`

✓ **Hidden Files** - Start with `.` (dot), use `ls -la` to see them

✓ **Root vs Root** - `/` is filesystem root; `root` is superuser

✓ **Root Privileges** - Many hacking tools require root access

✓ **Line Continuation** - When entering commands interactively with `cat >`, press `Ctrl+D` to exit and save

✓ **Combine Flags** - `ls -l -a` can be written as `ls -la`

---

## Quick Command Chains

```bash
# Find all config files in /etc
find /etc -type f -name *.conf

# Check if a service is running
ps aux | grep servicename

# List directory with hidden files
ls -la

# Navigate to home then show contents
cd ~ && ls -l

# Create directory and enter it
mkdir mynewdir && cd mynewdir
```

---

## Quick Start
Login as root → Open terminal → Use `pwd` to see location → Use `cd` to navigate → Use `ls -la` to explore → Use `man <command>` for detailed help

