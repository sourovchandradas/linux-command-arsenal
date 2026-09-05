# Automating Tasks with Job Scheduling

## Table of Contents

* [Introduction & Cron Architecture](#introduction--cron-architecture)
* [System Crontab Field Structure & Syntax](#system-crontab-field-structure--syntax)
* [Crontab Examples & Built-In Shortcuts](#crontab-examples--built-in-shortcuts)
* [Managing Startup Services with rc.d & Runlevels](#managing-startup-services-with-rcd--runlevels)
* [GUI Startup Management (rcconf)](#gui-startup-management-rcconf)
* [Key Tips & Source Text Gotchas](#key-tips--source-text-gotchas)
* [Quick Command Chains](#quick-command-chains)
* [Quick Start](#quick-start)

---

## Introduction & Cron Architecture

Linux administrative and security tasks (such as log rotation, system backups, and scheduled port scanning) rely on automated background execution.

```
┌─────────────────────────────────────────────────────────┐
│                      CROND DAEMON                       │
│     (Background daemon checking /etc/crontab constantly)│
└────────────────────────────┬────────────────────────────┘
                             │ Evaluates Schedules
┌────────────────────────────▼────────────────────────────┐
│                      CRON TABLE                         │
│   (/etc/crontab file defining time, user & commands)    │
└────────────────────────────┬────────────────────────────┘
                             │ Triggers Execution
┌────────────────────────────▼────────────────────────────┐
│                   TARGET SCRIPT / TASK                  │
│       (e.g., MySQLscanner.sh, systembackup.sh)          │
└─────────────────────────────────────────────────────────┘

```

| Component / Concept | Description & Security Impact |
| --- | --- |
| **`crond` Daemon** | Background daemon that monitors the system cron table and triggers scheduled commands. |
| **`crontab` File** | Configuration table located at `/etc/crontab` storing scheduled execution rules. |
| **Startup (`rc`) Scripts** | Bootup execution scripts executed by `init`/`initd` from `/etc/init.d/rc` to start services automatically. |

---

## System Crontab Field Structure & Syntax

The system-wide cron table (`/etc/crontab`) uses **7 mandatory fields**. The first five define the time schedule, the sixth defines the execution context (`USER`), and the seventh specifies the absolute file path to the executable command.

### Crontab Field Breakdown

| Field # | Time / Setting Unit | Valid Numerical Range | Notes & Syntax Rules |
| --- | --- | --- | --- |
| **1** | Minute | `0–59` | Exact minute of the hour. |
| **2** | Hour | `0–23` | 24-hour time notation (`13` = 1 PM). |
| **3** | Day of the Month (DOM) | `1–31` | Calendar day. |
| **4** | Month | `1–12` | Must be numeric (`3` = March; text names not allowed). |
| **5** | Day of the Week (DOW) | `0–7` | `0` and `7` both represent Sunday (`1` = Monday, `6` = Saturday). |
| **6** | User Context | `root`, `user`, etc. | Account privileges under which the command executes. |
| **7** | Command / Path | Absolute File Path | Must use full path (e.g., `/usr/share/MySQLscanner.sh`). |

### Wildcards & Syntax Operators

* **Asterisk (`*`):** Wildcard matching all possible values (read as "every" minute/day/month).
* **Dash (`-`):** Specifies an inclusive range of values (e.g., `1-5` for Monday through Friday).
* **Comma (`,`):** Specifies noncontiguous individual values (e.g., `15,30` for the 15th and 30th days; `0,6` for Sunday and Saturday).

---

## Crontab Examples & Built-In Shortcuts

### Scheduled Job Examples

```text
# Field Format Reference:
# M  H  DOM  MON  DOW  USER  COMMAND

# Run port scanner at 2:30 AM every Monday through Friday as root
30 2 * * 1-5 root /root/myscanningscript

# Run system backup at 2:00 AM every Sunday as user 'backup'
00 2 * * 0 backup /bin/systembackup.sh

# Run system backup at 2:00 AM on the 15th and 30th of every month
00 2 15,30 * * backup /root/systembackup.sh

# Run system backup every weeknight (Mon-Fri) at 11:00 PM (23:00)
00 23 * * 1-5 backup /root/systembackup.sh

# Run MySQL scanner daily at 9:00 AM as regular user
00 9 * * * user /usr/share/MySQLscanner.sh

# Run MySQL scanner at 2:00 AM on weekends (Sat/Sun) during summer (June-August)
00 2 * 6-8 0,6 user /usr/share/MySQLscanner.sh

```

### Built-In Macro Shortcuts

Special macros can replace the 5 numeric time fields in `crontab`:

```text
@midnight user /usr/share/MySQLscanner.sh

```

| Macro Shortcut | Equivalent Time Specification |
| --- | --- |
| `@yearly` / `@annually` | `0 0 1 1 *` (Once a year at midnight on Jan 1st) |
| `@monthly` | `0 0 1 * *` (Once a month at midnight on the 1st) |
| `@weekly` | `0 0 * * 0` (Once a week at midnight on Sunday) |
| `@daily` / `@midnight` | `0 0 * * *` (Once a day at midnight) |
| `@noon` | `0 12 * * *` (Once a day at 12:00 PM) |
| `@reboot` | Triggers execution immediately upon system startup |

---

## Managing Startup Services with rc.d & Runlevels

During system boot, after the kernel initializes, the `init` / `initd` daemon runs scripts inside `/etc/init.d/rc` according to the active **Runlevel**.

### Linux Runlevel Reference

| Runlevel | Operational Mode / State |
| --- | --- |
| **`0`** | Halt / Power down system |
| **`1`** | Single-user / Minimal recovery mode (network services disabled) |
| **`2 – 5`** | Multiuser modes (standard network operational state) |
| **`6`** | Reboot system |

### CLI Service Management (`update-rc.d`)

To configure services (such as PostgreSQL for Metasploit) to launch automatically at boot:

```bash
# 1. Verify if the target service is actively running
ps aux | grep postgresql

# 2. Configure service to start automatically across default runlevels at boot
update-rc.d postgresql defaults

# 3. Reboot the system to initialize configured startup services
reboot

# 4. Verify process execution post-reboot
ps aux | grep postgresql

```

---

## GUI Startup Management (rcconf)

For menu-driven startup script configuration:

```bash
# Install rcconf utility from Kali repositories
apt-get install rcconf

# Launch the interactive terminal GUI
rcconf

```

* **Controls:** Use `Up/Down Arrows` to navigate services, `Spacebar` to toggle startup status (`*` = enabled), `Tab` to navigate to **OK**, and `Enter` to save.

---

## Key Tips & Source Text Gotchas

* **Typo in Script Name (`MySQLsscanner.sh`)** — Page 177 text repeatedly lists `/usr/share/MySQLsscanner.sh` with a double "s", whereas the script created in Chapter 8 and referenced in Chapter 16 text is `MySQLscanner.sh`.
* **Inconsistent Backup Path** — Page 176 specifies `/bin/systembackup.sh` for the backup script, but on page 177 the subsequent crontab entries shift the path to `/root/systembackup.sh`.
* **OCR Artifact (`II` vs `||`)** — Listing 16-2 shows `test -x /usr/sbin/anacron II ...` using capital `I` characters instead of the standard bash logical OR operator `||`.
* **System Crontab vs User Crontab** — Editing `/etc/crontab` (system-wide) requires the 6th field (`USER`), whereas user-specific crontabs edited via `crontab -e` omit the username field.

---

## Quick Command Chains

```bash
# Open system crontab in standard text editor
leafpad /etc/crontab

# Edit user-specific crontab interactively
crontab -e

# Enable PostgreSQL to launch automatically at bootup and verify process
update-rc.d postgresql defaults && ps aux | grep postgresql

```

---

## Quick Start

Edit Schedule (`leafpad /etc/crontab`) $\rightarrow$ Format Time (`M H DOM MON DOW`) $\rightarrow$ Define User & Path (`root /path/to/script`) $\rightarrow$ Enable Boot Service (`update-rc.d <service> defaults`) $\rightarrow$ Verify Process (`ps aux | grep <service>`)
