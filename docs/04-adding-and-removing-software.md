# Adding and Removing Software

## Table of Contents

- [Introduction](#introduction)
- [Package Search and Inspection](#package-search-and-inspection)
- [Installing, Removing, and Purging Software](#installing-removing-and-purging-software)
- [Updating and Upgrading Packages](#updating-and-upgrading-packages)
- [Managing Repositories (/etc/apt/sources.list)](#managing-repositories-etcaptsourceslist)
- [GUI Package Managers](#gui-package-managers)
- [Cloning Software with Git](#cloning-software-with-git)
- [Key Tips](#key-tips)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

---

## Introduction

In Linux, software management involves installing, updating, and removing software packages—bundled archives containing applications, binaries, configuration scripts, and required dependencies. In Debian-based distributions like Kali Linux, software is managed primarily via the APT (Advanced Packaging Tool) suite (`apt-get`, `apt-cache`), repository config files (`/etc/apt/sources.list`), GUI package managers (`synaptic`), and GitHub (`git`).

---

## Package Search and Inspection

| Command | Description | Example |
| --- | --- | --- |
| `apt-cache search <keyword>` | Query local APT cache for packages matching a keyword | `apt-cache search snort` |

```bash
# Search repository cache for Snort Network Intrusion Detection System packages
apt-cache search snort

```

---

## Installing, Removing, and Purging Software

| Action | Command Syntax | Description |
| --- | --- | --- |
| Install | `apt-get install <package>` | Download and install package along with required dependencies |
| Remove | `apt-get remove <package>` | Uninstall software package while preserving configuration files |
| Purge | `apt-get purge <package>` | Uninstall software package and delete all accompanying configuration files |
| Auto-remove | `apt autoremove` | Remove orphaned dependency libraries no longer required by any application |

```bash
# Install Snort from default repositories
apt-get install snort

# Remove Snort but keep configuration files for future re-installation
apt-get remove snort

# Completely purge Snort along with its configuration files
apt-get purge snort

# Remove leftover orphaned dependencies (e.g., libdaq, oinkmaster)
apt autoremove

```

---

## Updating and Upgrading Packages

| Operation | Command Syntax | Purpose |
| --- | --- | --- |
| Update Index | `apt-get update` | Refresh local package database from remote repositories |
| Upgrade Packages | `apt-get upgrade` | Download and install newest versions of all installed packages |

```bash
# Refresh available package index list from online software repositories
apt-get update

# Upgrade all currently installed system packages to their latest versions
apt-get upgrade

```

---

## Managing Repositories (/etc/apt/sources.list)

| Category | Description |
| --- | --- |
| `main` | Fully supported open-source software |
| `universe` | Community-maintained open-source software |
| `multiverse` | Software restricted by copyright or legal issues |
| `restricted` | Proprietary device drivers |
| `backports` | Packages ported from later distribution releases |

```bash
# Open the repository sources file with a text editor
leafpad /etc/apt/sources.list

# Example third-party repository entries in sources.list:
# deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main
# deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu precise main
```

---

## GUI Package Managers

| GUI Manager | Purpose | Installation / Launch |
| --- | --- | --- |
| `synaptic` | Graphical APT frontend for searching, installing, and removing packages | `apt-get install synaptic` $\rightarrow$ `synaptic` |
| `gdebi` | Lightweight GUI tool for installing standalone `.deb` package files | `apt-get install gdebi` $\rightarrow$ `gdebi` |

```bash
# Install Synaptic Package Manager
apt-get install synaptic

# Launch Synaptic Package Manager from CLI
synaptic

```

---

## Cloning Software with Git

| Command Syntax | Description | Example |
| --- | --- | --- |
| `git clone <repository_url>` | Copy remote source code repository from GitHub to local machine | `git clone https://www.github.com/balle/bluediving.git` |

```bash
# Download software directly from GitHub when unavailable in APT repos
git clone https://www.github.com/balle/bluediving.git

# Verify cloned repository folder in current working directory
ls -l

```

---

## Key Tips

* **`apt-get` vs `apt`** — While `apt` provides simpler output, `apt-get` offers broader functionality and script reliability.
* **`remove` vs `purge`** — Use `apt-get remove` to retain settings for future reinstalls; use `apt-get purge` for a complete wipe.
* **`update` vs `upgrade`** — `apt-get update` only refreshes package lists (indexes); `apt-get upgrade` actually installs the updated software binaries.
* **Avoid Unstable Repositories** — Do not add `testing`, `experimental`, or `unstable` repos to `/etc/apt/sources.list` as they can break Kali binaries.
* **Orphaned Dependencies** — Uninstalling a package leaves behind installed dependencies; run `apt autoremove` to reclaim disk space.

---

## Quick Command Chains

```bash
# Update repository index and perform full package upgrade sequentially
apt-get update && apt-get upgrade -y

# Purge software package and remove leftover orphan dependencies together
apt-get purge snort -y && apt autoremove -y

```

---

## Quick Start

Search cache (`apt-cache search`) $\rightarrow$ Fetch latest index (`apt-get update`) $\rightarrow$ Install package (`apt-get install`) $\rightarrow$ Clean orphans (`apt autoremove`) $\rightarrow$ Clone custom tools (`git clone`)
