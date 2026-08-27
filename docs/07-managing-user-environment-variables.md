# Managing User Environment Variables

## Table of Contents

- [Introduction](#introduction)
- [Viewing and Filtering Variables (env, set, grep)](#viewing-and-filtering-variables-env-set-grep)
- [Modifying and Exporting Variables (export, HISTSIZE)](#modifying-and-exporting-variables-export-histsize)
- [Customizing the Shell Prompt (PS1)](#customizing-the-shell-prompt-ps1)
- [Managing the PATH Variable](#managing-the-path-variable)
- [Creating and Deleting Custom Variables (unset)](#creating-and-deleting-custom-variables-unset)
- [Key Tips](#key-tips)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

---

## Introduction

Environment variables are process-wide key-value pairs (`KEY=value`) built into Linux that define terminal appearance, behavior, search paths, and system settings. In Bash, variables fall into two categories: **Environment Variables** (system-wide, uppercase, inherited by child processes) and **Shell Variables** (local to the current shell session). Managing variables allows system customization, execution optimization, and operational stealth.

---

## Viewing and Filtering Variables (env, set, grep)

### Inspection Commands

| Command | Target Scope | Description |
| --- | --- | --- |
| `env` | Environment Variables | Displays system-wide default environment variables (UPPERCASE) |
| `set` | All Variables & Functions | Displays environment variables, shell variables, local settings, and aliases |
| `set \| more` | All Variables (Paginated) | Displays full variable list page-by-page (`Enter` to scroll, `q` to exit) |
| `set \| grep <VAR>` | Specific Variable | Filters environment and shell list for a target variable name |

```bash
# View standard system environment variables
env

# View all variables with pagination
set | more

# Check current history buffer limit (HISTSIZE)
set | grep HISTSIZE

```

---

## Modifying and Exporting Variables (export, HISTSIZE)

Variable changes made directly in the terminal are session-bound and expire when the shell closes. Using `export` propagates the variable to child shells and processes.

### Anti-Forensic Command History Control (`HISTSIZE`)

| Command | Action | Operational Impact |
| --- | --- | --- |
| `HISTSIZE=0` | Set history memory buffer to zero | Prevents terminal from saving typed command logs (Stealth) |
| `export HISTSIZE` | Export setting to child shells | Enforces zero command logging across subshells |
| `HISTSIZE=1000` | Reset buffer to default | Restores standard command history retention (1000 entries) |

```bash
# Backup current variable values before modification
echo $HISTSIZE > ~/valueofHISTSIZE.txt
set > ~/environment_backup.txt

# Disable command history for stealth in current session
HISTSIZE=0

# Export modified variable so child processes inherit it
export HISTSIZE

# Restore history size to default
HISTSIZE=1000 && export HISTSIZE

```

---

## Customizing the Shell Prompt (PS1)

The `PS1` variable controls the structure and prompt text displayed in the shell interface.

### Common `PS1` Placeholders

| Token | Represents | Output Example |
| --- | --- | --- |
| `\u` | Active Username | `root` |
| `\h` | System Hostname | `kali` |
| `\w` | Full Working Directory Path | `/usr/bin` |
| `\W` | Base Directory Name | `bin` |

```bash
# View current prompt configuration
echo $PS1

# Change prompt text for current session
PS1="World's Best Hacker: #"

# Windows CMD Prompt Emulation (Displays current directory)
export PS1='C:\w> '

# Revert to standard Kali prompt format
export PS1='\u@\h:\w\$ '

```

---

## Managing the PATH Variable

The `$PATH` variable contains an ordered, colon-separated list of directories (`/usr/bin:/sbin:...`) where Linux searches for executable binary files when a command is entered.

### Correct vs. Incorrect PATH Configuration

| Method | Command Syntax | Resulting Behavior |
| --- | --- | --- |
| **Append (Correct)** | `PATH=$PATH:/root/newtool` | Preserves system paths and **appends** new directory to search list |
| **Overwrite (Fatal)** | `PATH=/root/newtool` | **Wipes** system binary paths; causes `command not found` for `ls`, `cat`, etc. |

```bash
# Display existing binary search paths
echo $PATH

# Safe: Append new custom tool directory to PATH
PATH=$PATH:/root/newhackingtool

# Export new PATH globally across subshells
export PATH

# Recovery fix if PATH was accidentally overwritten
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

```

---

## Creating and Deleting Custom Variables (unset)

Users can declare custom shell variables for command shortcuts, paths, or scripting data.

| Action | Command Syntax | Description |
| --- | --- | --- |
| **Declare** | `VARNAME="Value"` | Assigns string/value (No spaces around `=`) |
| **Reference** | `echo $VARNAME` | Prepend `$` to extract/read variable contents |
| **Export** | `export VARNAME` | Promotes variable to global environment variable |
| **Remove** | `unset VARNAME` | Deletes variable and clears memory allocation |

```bash
# Create custom user variable
MYNEWVARIABLE="Hacking is the most valuable skill set in the 21st century"

# Read variable value (must use '$' sign)
echo $MYNEWVARIABLE

# Export custom variable to subshells
export MYNEWVARIABLE

# Remove custom variable permanently from shell memory
unset MYNEWVARIABLE

```

---

## Key Tips

* **Dollar Sign (`$`) Value Extraction** — Omit `$` when assigning a variable (`VAR=value`), but always include `$` when reading or printing its content (`echo $VAR`).
* **Never Overwrite `$PATH**` — Always use `PATH=$PATH:/new/path`. Forgetting `$PATH:` strips Linux of default binary paths (`/bin`, `/usr/bin`), breaking basic utilities (`ls`, `grep`, `cd`).
* **Session vs. Permanent Persistence** — Terminal variable modifications expire on shell exit. To make changes permanent across reboots, add declarations (e.g., `export PATH=$PATH:/dir`) to `~/.bashrc` or `~/.profile`.
* **Quotes for Spaces** — Encapsulate values containing spaces in quotes (e.g., `MYVAR="value with spaces"`).

---

## Quick Command Chains

```bash
# Backup environment, disable command history logging, and export stealth mode
set > ~/.env_bak && HISTSIZE=0 && export HISTSIZE

# Add custom binaries directory to PATH and export immediately
PATH=$PATH:/opt/custom_tools && export PATH

```

---

## Quick Start

View env (`env`) $\rightarrow$ Check variable content (`echo $VAR`) $\rightarrow$ Modify variable (`VAR=value`) $\rightarrow$ Export to subshells (`export VAR`) $\rightarrow$ Delete variable (`unset VAR`)
