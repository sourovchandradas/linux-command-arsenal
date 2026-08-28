# Bash Scripting

## Table of Contents

- [Introduction](#introduction)
- [Core Scripting Elements and Execution](#core-scripting-elements-and-execution)
- [Handling User Input and Variables](#handling-user-input-and-variables)
- [Practical Hacker Scripting: Network Port Scanners](#practical-hacker-scripting-network-port-scanners)
- [Essential Built-in Bash Commands](#essential-built-in-bash-commands)
- [Key Tips](#key-tips)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

---

## Introduction

Bash scripting automates terminal commands, combines multiple utilities, and creates custom security tools. A shell serves as the direct interface between the user and the operating system kernel. The Bourne-again shell (`bash`) is the default interpreter across most Unix/Linux distributions, supporting both standard CLI applications and built-in scripting constructs.

---

## Core Scripting Elements and Execution

### Essential Script Components

| Component | Syntax Example | Function |
| --- | --- | --- |
| **Shebang** | `#!/bin/bash` | Specifies the path to the interpreter used to run the script |
| **Comment** | `# This is a comment` | Explanatory note ignored by the Bash interpreter |
| **Print Output** | `echo "Message"` | Displays text strings or variable contents to stdout |
| **File Extension** | `script.sh` | Standard convention for shell scripts (optional, non-enforced) |

### Script Execution Workflow

```bash
# 1. Create script file using a text editor
mousepad FirstScript.sh

# 2. Add shebang and commands
#!/bin/bash
# Basic test script
echo "Hello, Hackers-Arise!"

# 3. Grant execute permissions (chmod 755 or chmod +x)
chmod 755 FirstScript.sh

# 4. Execute script from the current directory using relative pathing
./FirstScript.sh

```

---

## Handling User Input and Variables

Variables store dynamic data in memory. User input is captured using the `read` command and referenced using the `$` prefix.

| Operation | Command Syntax | Description |
| --- | --- | --- |
| **Capture Input** | `read VAR_NAME` | Reads standard input from the keyboard into `VAR_NAME` |
| **Reference Variable** | `$VAR_NAME` | Extracts the value stored inside `VAR_NAME` |
| **Prompt and Read** | `echo "Prompt:" && read VAR` | Displays prompt text before waiting for user entry |

```bash
#!/bin/bash
# User input and variable demonstration

echo "What is your name?"
read name

echo "What chapter are you on in Linux Basics for Hackers?"
read chapter

# Reference captured input using the $ operator
echo "Welcome $name to Chapter$chapter of Linux Basics for Hackers!"

```

---

## Practical Hacker Scripting: Network Port Scanners

Combining `nmap`, input redirection (`>/dev/null`), output formatting (`-oG`), and text filters (`grep`) enables automated target discovery.

### Basic Local Subnet Scanner

```bash
#!/bin/bash
# Scan LAN for active MySQL databases (Port 3306)

# Perform TCP Connect scan (-sT), suppress stdout, and output in grepable format (-oG)
nmap -sT 192.168.181.0/24 -p 3306 >/dev/null -oG MySQLscan

# Extract hosts with open ports into a refined result file
cat MySQLscan | grep open > MySQLscan2

# Display results
cat MySQLscan2

```

### Interactive Dynamic Scanner

```bash
#!/bin/bash
# Interactive multi-target port scanner

echo "Enter the starting IP address : "
read FirstIP

echo "Enter the last octet of the last IP address : "
read LastOctetIP

echo "Enter the port number you want to scan for : "
read port

# Run Nmap across defined range ($FirstIP-$LastOctetIP) for target$port
nmap -sT $FirstIP-$LastOctetIP -p$port >/dev/null -oG MySQLscan

# Filter and display open ports
cat MySQLscan | grep open > MySQLscan2
cat MySQLscan2

```

---

## Essential Built-in Bash Commands

| Command | Function |
| --- | --- |
| `:` | Returns 0 or true exit status |
| `.` | Executes a shell script within the current shell context |
| `bg` | Sends a suspended job to the background |
| `fg` | Brings a background job to the foreground |
| `break` | Exits the active loop construct |
| `continue` | Skips to the next iteration of the active loop |
| `cd` | Changes the active working directory |
| `echo` | Prints arguments to standard output |
| `eval` | Evaluates combined arguments as a shell command |
| `exec` | Executes a command, replacing the current shell process |
| `exit` | Terminates the active shell session |
| `export` | Marks variables/functions to be inherited by child processes |
| `getopts` | Parses positional parameters and flags in scripts |
| `jobs` | Lists active background processes running in current shell |
| `pwd` | Displays absolute path of current working directory |
| `read` | Reads a line from standard input into a variable |
| `readonly` | Locks a variable as read-only (cannot be reassigned or unset) |
| `set` | Sets shell options or lists all shell variables |
| `shift` | Shifts positional parameters (`$1`, `$2`) to the left |
| `test` / `[[` | Evaluates conditional expressions and tests files |
| `times` | Displays accumulated process execution times |
| `trap` | Intercepts system signals (`SIGINT`, etc.) to execute custom handlers |
| `type` | Displays how a command name is interpreted (builtin, alias, binary) |
| `umask` | Sets default file creation permission mask |
| `unset` | Removes variable or function definitions from memory |
| `wait` | Suspends execution until background processes complete |

---

## Key Tips

* **Always Specify Shebang** — Place `#!/bin/bash` on line 1 to prevent execution errors if the script runs under a different default shell (e.g., `sh` or `zsh`).
* **Explicit Execution (`./`)** — Always use `./script.sh` to execute files in the current directory; Linux excludes the current working directory from `$PATH` by default for security.
* **Variable Prefixes** — Do not use `$` when assigning variables (`read TargetIP` or `PORT=80`), only when referencing them (`$TargetIP` or `$PORT`).
* **Silent Execution** — Redirect irrelevant command output to `/dev/null` (`command > /dev/null`) to keep terminal output clean and focused.

---

## Quick Command Chains

```bash
# Create script, set executable permissions, and open in editor
touch scanner.sh && chmod 755 scanner.sh && mousepad scanner.sh

# Run script, filter output for specific open ports, and save log
./scanner.sh | grep "3306/open" | tee active_db_hosts.txt

```

---

## Quick Start

Create script (`nano script.sh`) $\rightarrow$ Add Shebang (`#!/bin/bash`) $\rightarrow$ Make executable (`chmod 755 script.sh`) $\rightarrow$ Run locally (`./script.sh`)

