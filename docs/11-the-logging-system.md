# The Logging System

## Table of Contents

- [Introduction](#introduction)
- [The rsyslog Logging Daemon](#the-rsyslog-logging-daemon)
- [rsyslog Rule Configuration (Facility, Priority, Action)](#rsyslog-rule-configuration-facility-priority-action)
- [Log Rotation with logrotate](#log-rotation-with-logrotate)
- [Anti-Forensics and Anti-Logging Techniques](#anti-forensics-and-anti-logging-techniques)
- [Key Tips](#key-tips)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

---

## Introduction

Log files store records of system events, application execution, errors, and security alerts. For system administrators and security analysts, log management is essential for incident response, troubleshooting, and forensic analysis. For penetration testers and offensive security operators, understanding logging architecture is critical for gathering intelligence on target system activities, assessing detection capabilities, and safely clearing activity traces or suppressing log generation.

---

## The rsyslog Logging Daemon

Linux uses logging daemons derived from traditional `syslogd`. While other Linux distributions may use variants like `syslog-ng`, Kali Linux (Debian-based) utilizes `rsyslog` by default.

### Key System Paths

| File / Directory Path | Function |
| --- | --- |
| `/etc/rsyslog.conf` | Primary `rsyslog` daemon configuration file |
| `/etc/rsyslog.d/` | Directory containing modular application-specific logging rules |
| `/etc/default/rsyslog` | Startup environment variables and parameters for the daemon |
| `/etc/init.d/rsyslog` | Init service control script |
| `/var/log/` | Standard system directory where generated log files reside |

### Core Modules in rsyslog.conf

```text
module(load="imuxsock") # Provides support for local system socket logging
module(load="imklog")   # Provides kernel logging support (dmesg/kernel events)
#module(load="immark")  # Provides --MARK-- timestamp capability for log auditing
#module(load="imudp")   # UDP syslog reception (Port 514) - Default Disabled
#module(load="imtcp")   # TCP syslog reception (Port 514) - Default Disabled

```

---

## rsyslog Rule Configuration (Facility, Priority, Action)

Logging behavior is defined under the `RULES` section (~line 55) of `/etc/rsyslog.conf`.

### Rule Syntax Structure

```text
facility.priority           action

```

### 1. Facility Keywords

Refers to the system component or program generating the log event.

| Facility Code | System Subsystem / Source |
| --- | --- |
| `auth`, `authpriv` | Security and authentication/authorization events |
| `cron` | Scheduled task clock daemons |
| `daemon` | Background system services and daemons |
| `kern` | Kernel-generated messages |
| `lpr` | Printing system messages |
| `mail` | Mail server system events |
| `user` | Generic user-level software messages |
| `*` | **Wildcard:** Applies rule to all facilities |

### 2. Priority Levels (Severity Hierarchy)

Specifying a priority code instructs `rsyslog` to log events at that level **and all higher priorities**.

```text
[Lowest Severity]  debug -> info -> notice -> warning -> err -> crit -> alert -> emerg  [Highest Severity]

```

| Priority Code | Description |
| --- | --- |
| `debug` | Detailed debugging information |
| `info` | Informational operational messages |
| `notice` | Normal but significant condition |
| `warning` | Warning conditions |
| `err` | Error conditions |
| `crit` | Critical hardware/software conditions |
| `alert` | Immediate action required |
| `emerg` | System is unusable (Emergency broadcast) |
| `*` | **Wildcard:** Matches all priority levels |
| `none` | Disables logging for the specified facility |

> **Deprecated Codes:** `warn` (use `warning`), `error` (use `err`), and `panic` (use `emerg`) are deprecated and should no longer be used.

### 3. Action Destinations

Defines where event messages are written.

* **File Paths:** Direct file output (e.g., `/var/log/auth.log`). Preceding a path with a minus sign (e.g., `-/var/log/syslog`) disables immediate disk buffer flushing for improved performance.
* **User Broadcast:** Sends real-time terminal notices to active users (e.g., `:omusmsg:*`).

### Rule Examples Breakdown

| Configuration Syntax Rule | Operational Behavior |
| --- | --- |
| `auth,authpriv.* /var/log/auth.log` | Logs all priorities (`*`) for security/auth facilities to `/var/log/auth.log` |
| `mail.* /var/log/mail` | Logs all mail facility messages to `/var/log/mail` |
| `kern.crit /var/log/kernel` | Logs kernel messages of `crit`, `alert`, and `emerg` priorities to `/var/log/kernel` |
| `*.emerg :omusmsg:*` | Broadcasts emergency messages across all facilities to all logged-in terminals |

---

## Log Rotation with logrotate

To prevent log files from exhausting hard drive storage, the system automates archival, compression, and purging via `logrotate` managed by `cron`. Consult `man logrotate` for advanced custom directive parameters.

### Configuration Directives (`/etc/logrotate.conf`)

| Directive Flag | Description / Function |
| --- | --- |
| `weekly` / `daily` / `monthly` | Sets the time interval unit for rotation execution |
| `rotate 4` | Retains 4 weeks/cycles of archived log backups before deleting the oldest |
| `create` | Generates a new, empty log file immediately after rotating old logs |
| `compress` | Compresses rotated archives using `gzip` (saves disk space) |
| `include /etc/logrotate.d` | Imports additional rotation configurations for installed packages |

### Rotation Numerical Chaining

When a rotation occurs, log files are renamed sequentially. The oldest log beyond the `rotate` limit is permanently dropped:

```text
auth.log (Active) -> auth.log.1 -> auth.log.2 -> auth.log.3 -> auth.log.4 -> [Deleted]

```

```bash
# View all active and rotated authorization logs
ls /var/log/auth.log*

```

---

## Anti-Forensics and Anti-Logging Techniques

Offensive security operators use anti-forensic measures to prevent leaving actionable artifacts in target system logs.

### 1. Limitations of Manual Line Deletion

Deleting specific log lines using text editors or standard `rm` commands is unreliable for stealth operations:

* Leaves unnatural timestamp gaps that alert forensic investigators.
* Standard filesystem deletion only unlinks file pointers; raw data blocks remain intact on disk and are recoverable using forensic carving tools.

### 2. Secure Overwriting with `shred`

The `shred` utility prevents forensic recovery by repeatedly overwriting target files with random data patterns (4 passes by default).

| `shred` Parameter | Function |
| --- | --- |
| `-f`, `--force` | Modifies file permissions if necessary to grant write access |
| `-n <N>` | Overwrites target file $N$ times instead of the default 4 |

```bash
# Securely overwrite all active and rotated auth logs 10 times with random patterns
shred -f -n 10 /var/log/auth.log.*

```

> **Result:** Opening a shredded file presents unrecoverable, scrambled binary noise.

### 3. Disabling Logging Daemons

To halt log generation during active operations, stop the logging daemon (`root` privileges required).

**Generic Service Control Syntax:**
`service <service_name> start|stop|restart`

```bash
# Stop the rsyslog service completely
service rsyslog stop

# Verify service termination status
service rsyslog status

```

---

## Key Tips

* **Asynchronous Buffer Writing (`-`)** — Preceding a log path with `-` in `rsyslog.conf` (e.g., `-/var/log/syslog`) buffers writes in memory to optimize I/O performance.
* **Priority Cascade Rule** — Setting a priority rule like `kern.err` captures `err`, `crit`, `alert`, and `emerg`. To capture *only* errors, explicit suppression flags must be applied.
* **Shredding Rotated Chains** — Always append wildcards (`auth.log.*`) when clearing logs with `shred` to wipe both active files and historical `logrotate` archives.
* **Time Overhead of `shred**` — Higher pass counts (`-n 20`) on large log files significantly increase CPU and disk I/O time.

---

## Quick Command Chains

```bash
# Locate all rsyslog system configuration and service files
locate rsyslog

# Perform anti-forensic shredding across all auth logs and immediately halt logging
shred -f -n 10 /var/log/auth.log.* && service rsyslog stop

# Search rotated log archives for historical authentication attempts
grep "Failed password" /var/log/auth.log*

```

---

## Quick Start

Locate Rules (`/etc/rsyslog.conf`) $\rightarrow$ Check Rotation Policy (`/etc/logrotate.conf`) $\rightarrow$ Disable Daemon (`service rsyslog stop`) $\rightarrow$ Secure Wipe Logs (`shred -f -n 10 /var/log/auth.log.*`)
