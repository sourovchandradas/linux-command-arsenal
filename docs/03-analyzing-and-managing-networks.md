# Analyzing and Managing Networks

## Table of Contents

* [Introduction](https://www.google.com/search?q=%23introduction)
* [Analyzing Network Interfaces](https://www.google.com/search?q=%23analyzing-network-interfaces)
* [Configuring IP and Network Parameters](https://www.google.com/search?q=%23configuring-ip-and-network-parameters)
* [Spoofing MAC Addresses](https://www.google.com/search?q=%23spoofing-mac-addresses)
* [DHCP IP Reassignment](https://www.google.com/search?q=%23dhcp-ip-reassignment)
* [DNS Reconnaissance and Configuration](https://www.google.com/search?q=%23dns-reconnaissance-and-configuration)
* [Local IP Mapping with /etc/hosts](https://www.google.com/search?q=%23local-ip-mapping-with-etchosts)
* [Key Tips](https://www.google.com/search?q=%23key-tips)
* [Quick Command Chains](https://www.google.com/search?q=%23quick-command-chains)
* [Quick Start](https://www.google.com/search?q=%23quick-start)

---

## Introduction

In Linux, mastering network management tools is essential for penetration testing and system administration. Key tasks include analyzing network interfaces, assigning static IPs, spoofing MAC addresses, requesting dynamic IPs via DHCP, conducting DNS reconnaissance, and customizing local name resolution.

---

## Analyzing Network Interfaces

| Command | Description | Example |
| --- | --- | --- |
| `ifconfig` | Query active network interfaces (`eth0`, `lo`, `wlan0`) | `ifconfig` |
| `iwconfig` | Query wireless network interfaces (IEEE standard, Mode, Power) | `iwconfig` |

```bash
# View active network interfaces and assigned IPs
ifconfig

# View wireless adapter settings (wireless standards, Mode, Tx-Power)
iwconfig

```

---

## Configuring IP and Network Parameters

| Command Syntax | Description | Example |
| --- | --- | --- |
| `ifconfig <iface> <ip>` | Reassign IP address to an interface | `ifconfig eth0 192.168.181.115` |
| `ifconfig <iface> <ip> netmask <mask> broadcast <bcast>` | Reassign IP, netmask, and broadcast address simultaneously | `ifconfig eth0 192.168.181.115 netmask 255.255.0.0 broadcast 192.168.1.255` |

```bash
# Temporarily assign a new IP address
ifconfig eth0 192.168.181.115

# Change IP address, network mask, and broadcast address together
ifconfig eth0 192.168.181.115 netmask 255.255.0.0 broadcast 192.168.1.255

```

---

## Spoofing MAC Addresses

| Action | Command | Purpose |
| --- | --- | --- |
| 1. Disable Interface | `ifconfig <iface> down` | Take down the interface before changing hardware properties |
| 2. Change MAC Address | `ifconfig <iface> hw ether <mac>` | Spoof hardware MAC address (`HWaddr`) |
| 3. Enable Interface | `ifconfig <iface> up` | Bring the interface back online with the spoofed MAC |

```bash
# Full command sequence to spoof a MAC address on eth0
ifconfig eth0 down
ifconfig eth0 hw ether 00:11:22:33:44:55
ifconfig eth0 up

```

---

## DHCP IP Reassignment

| Command | Description | Example |
| --- | --- | --- |
| `dhclient <iface>` | Request a new IP address from the DHCP daemon (`dhcpd`) | `dhclient eth0` |

```bash
# Request a fresh IP lease via DHCPDISCOVER / DHCPOFFER workflow
dhclient eth0

```

---

## DNS Reconnaissance and Configuration

| File / Utility | Description | Example |
| --- | --- | --- |
| `dig <domain> ns` | Query Domain Name Server (NS) records and server IP | `dig hackers-arise.com ns` |
| `dig <domain> mx` | Query Mail Exchange (MX) server records | `dig hackers-arise.com mx` |
| `/etc/resolv.conf` | Plaintext configuration file specifying active DNS nameservers | `cat /etc/resolv.conf` |
| `echo "<entry>" > <file>` | Replace DNS server configuration directly from CLI | `echo "nameserver 8.8.8.8" > /etc/resolv.conf` |

```bash
# Perform DNS reconnaissance on target nameservers and mail servers
dig hackers-arise.com ns
dig hackers-arise.com mx

# Direct DNS queries to Google's public DNS server
echo "nameserver 8.8.8.8" > /etc/resolv.conf

```

---

## Local IP Mapping with /etc/hosts

| File | Purpose | Syntax Format |
| --- | --- | --- |
| `/etc/hosts` | Maps IP addresses to domain names locally (overrides DNS lookup) | `<IP_ADDRESS>[Tab]<DOMAIN_NAME>` |

```bash
# View current local DNS resolution overrides
cat /etc/hosts

# Example host mapping in /etc/hosts (press Tab between IP and domain):
# 192.168.181.131	bankofamerica.com

```

---

## Key Tips

* **Tab Separation in `/etc/hosts**` — Always press `Tab` between the IP address and domain name inside `/etc/hosts` (do not use spaces).
* **Interface Down Required** — Always take down the interface (`ifconfig eth0 down`) before modifying its MAC address.
* **DHCP Overwrites `/etc/resolv.conf**` — Renewing a DHCP IP lease using `dhclient` will overwrite manual changes in `/etc/resolv.conf`.
* **Wireless Modes in `iwconfig**` — Cracking wireless networks requires switching adapters from `Managed` mode to `Monitor` or `Promiscuous` mode.
* **Silent Success in Linux** — Successful execution of `ifconfig` configuration commands yields no terminal output and returns straight to the prompt.

---

## Quick Command Chains

```bash
# Spoof MAC address and request a fresh DHCP lease sequentially
ifconfig eth0 down && ifconfig eth0 hw ether 00:11:22:33:44:55 && ifconfig eth0 up && dhclient eth0

# Overwrite DNS server and run a DNS lookup check
echo "nameserver 8.8.8.8" > /etc/resolv.conf && dig hackers-arise.com ns

```

---

## Quick Start

Open terminal $\rightarrow$ Query interfaces with `ifconfig`/`iwconfig` $\rightarrow$ Spoof MAC address with `ifconfig down/hw/up` $\rightarrow$ Reset network lease using `dhclient` $\rightarrow$ Perform DNS reconnaissance with `dig` $\rightarrow$ Customize mappings in `/etc/resolv.conf` or `/etc/hosts`
