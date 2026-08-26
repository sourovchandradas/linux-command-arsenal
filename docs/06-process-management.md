# Process Management

## Table of Contents

- [Introduction](#introduction)
- [Viewing and Filtering Processes (ps, grep, top)](#viewing-and-filtering-processes-ps-grep-top)
- [Managing Process Priority (nice and renice)](#managing-process-priority-nice-and-renice)
- [Terminating Processes (kill and killall)](#terminating-processes-kill-and-killall)
- [Foreground and Background Execution (&, bg, fg)](#foreground-and-background-execution--bg-fg)
- [Scheduling Jobs with at](#scheduling-jobs-with-at)
- [Key Tips](#key-tips)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

---

## Introduction

In Linux, a process is an active program executing in memory and utilizing system resources (CPU, RAM). The Linux kernel assigns a unique Process ID (PID) sequentially to every running task. System administrators and security testers must monitor, prioritize, background, terminate, and schedule processes to maintain optimal system performance and control target environments.

---

## Viewing and Filtering Processes (ps, grep, top)

### Process Inspection Commands

| Tool / Command | Usage | Description |
| --- | --- | --- |
| `ps` | `ps` | View active processes initiated in the current shell session |
| `ps aux` | `ps aux` | Display detailed system-wide snapshot of all user and background processes |
| `ps aux \| grep <name>` | `ps aux \| grep msfconsole` | Filter system processes for a specific keyword or application |
| `top` | `top` | Launch dynamic real-time resource monitor sorted by heaviest usage |

### Understanding `ps aux` Output Fields

| Column | Full Name | Description |
| --- | --- | --- |
| `USER` | User Identity | Account/owner that launched the process |
| `PID` | Process ID | Unique numerical identifier assigned by the Linux kernel |
| `%CPU` | CPU Consumption | Percentage of processing unit currently consumed |
| `%MEM` | Memory Usage | Percentage of physical system RAM consumed |
| `COMMAND` | Executable Name | Full command path or binary invoked |

```bash
# View all processes running on the system
ps aux

# Search for Metasploit Framework process PID and resource usage
ps aux | grep msfconsole

# Launch real-time process monitoring interface
top

```

### Interactive `top` Shortcut Keys

| Key | Action |
| --- | --- |
| `H` or `?` | Open interactive help menu |
| `R` | Renice a process directly within the interface |
| `K` | Kill a targeted process PID directly |
| `Q` | Exit the `top` monitor |

---

## Managing Process Priority (nice and renice)

Process priority (niceness) dictates how kernel CPU scheduling treats competing processes.

* **Priority Scale:** Range spans from **$-20$** (Highest Priority / Least "Nice" to others) to **$+19$** (Lowest Priority / Most "Nice" to others).
* **Default Value:** `0` for newly spawned processes.

$$\text{Highest Priority (-20)} \longleftrightarrow \text{Default (0)} \longleftrightarrow \text{Lowest Priority (+19)}$$

### Priority Allocation Tools

| Command | Target | Adjustment Type | Privilege Requirement | Syntax Example |
| --- | --- | --- | --- | --- |
| `nice` | Binary Path | Relative increment/decrement (`-n`) | Negative values require `root` | `nice -n -10 /bin/slowprocess` |
| `renice` | Running PID | Absolute nice value setting | Negative values require `root` | `renice 19 6996` |

```bash
# Launch a new process with higher CPU priority (nice value lowered by 10)
nice -n -10 /bin/slowprocess

# Launch a process with lower priority to preserve background resources
nice -n 10 /bin/slowprocess

# Change priority of an ALREADY running process (PID 6996) to lowest priority (19)
renice 19 6996

```

---

## Terminating Processes (kill and killall)

Unresponsive, resource-heavy, or rogue processes are terminated using Linux signals.

### Commonly Used Kill Signals

| Signal Name | Option Number | Type | Description |
| --- | --- | --- | --- |
| `SIGHUP` | `-1` | Hangup | Restarts the target process keeping the same PID |
| `SIGINT` | `-2` | Interrupt | Weak termination request (Equivalent to `CTRL+C`) |
| `SIGQUIT` | `-3` | Quit | Core dump termination (Saves memory state to disk) |
| `SIGTERM` | `-15` | Terminate | Standard graceful termination (Default if no signal flag provided) |
| `SIGKILL` | `-9` | Kill | Absolute forceful stop; sends process resources to `/dev/null` |

```bash
# Restart a process using SIGHUP (PID 6996)
kill -1 6996

# Gracefully terminate a process (default SIGTERM)
kill 6996

# Forcefully kill an unresponsive process using SIGKILL
kill -9 6996

# Terminate all instances of a process by process NAME instead of PID
killall -9 rogueprocess

```

---

## Foreground and Background Execution (&, bg, fg)

Running long-duration tasks in the background prevents shell lockup and frees the terminal interface.

| Control Method | Command / Operator | Action |
| --- | --- | --- |
| **Launch in Background** | `&` | Append to command end (`command &`) to keep prompt free |
| **Send to Background** | `bg <PID>` | Resume a paused background task |
| **Bring to Foreground** | `fg <PID>` | Bring background process back to standard active terminal |

```bash
# Launch text editor in background to keep shell operational
leafpad newscript &

# Bring background task (PID 1234) back to active foreground shell
fg 1234

# Resume a suspended job in the background
bg 1234

```

---

## Scheduling Jobs with at

The `atd` daemon schedules non-recurring tasks to execute once at a specified future timestamp.

### Common `at` Time Formats

| Time Argument Syntax | Scheduled Execution Time |
| --- | --- |
| `at 7:20pm` | 7:20 PM on current date |
| `at 7:20pm June 25` | 7:20 PM on June 25 |
| `at noon` | 12:00 PM on current date |
| `at now + 20 minutes` | 20 minutes from current execution time |
| `at now + 10 hours` | 10 hours from current execution time |
| `at now + 5 days` | 5 days from current date |
| `at now + 3 weeks` | 3 weeks from current date |
| `at 7:20pm 06/25/2026` | 7:20 PM on June 25, 2026 |

### Interactive Job Scheduling Workflow

```bash
# 1. Define execution timestamp (Enters interactive 'at>' shell)
at 7:20am

# 2. Enter script/command to run at specified time
at> /root/myscanningscript

# 3. Press CTRL+D to save and submit the scheduled job

```

---

## Key Tips

* **PID Reliance** — Commands like `kill`, `renice`, and `fg` strictly require the numerical **PID**, whereas `killall` operates on the executable **Name**.
* **Root Privilege for High Priority** — Standard users can only increase niceness (lower priority). Assigning negative nice values ($-1$ to $-20$) requires `root` rights.
* **`kill -9` Safety** — Reserve `SIGKILL (-9)` for stubborn/rogue binaries because it does not allow applications to run cleanup routines before closing.
* **Saving Shell Space** — Always append `&` when opening GUI applications (e.g., `leafpad file &`, `wireshark &`) from CLI to avoid locking up your active terminal shell.

---

## Quick Command Chains

```bash
# Locate target application PID and immediately force terminate it
ps aux | grep rogueprocess | awk '{print $2}' | xargs kill -9

# Launch reconnaissance script in background with elevated CPU priority
nice -n -5 /root/recon.sh &

```

---

## Quick Start

Check active processes (`ps aux`) $\rightarrow$ Filter specific PID (`ps aux | grep <name>`) $\rightarrow$ Adjust execution priority (`renice`) $\rightarrow$ Background task (`bg` / `&`) $\rightarrow$ Terminate stuck processes (`kill -9 <PID>`)
