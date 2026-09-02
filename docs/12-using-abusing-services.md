# Using and Abusing Services

## Table of Contents

* [Introduction](#introduction)
* [Linux Service Management Syntax](#linux-service-management-syntax)
* [Apache Web Server (HTTP)](#apache-web-server-http)
* [OpenSSH & The Raspberry Spy Pi](#openssh--the-raspberry-spy-pi)
* [MySQL / MariaDB Database Exploitation](#mysql--mariadb-database-exploitation)
* [PostgreSQL with Metasploit](#postgresql-with-metasploit)
* [Key Tips](#key-tips)
* [Quick Command Chains](#quick-command-chains)
* [Quick Start](#quick-start)

---

## Introduction

A service is a background application waiting for interaction. Linux systems include dozens of preinstalled services. For security testing and offensive operations, four critical services are central: Apache Web Server (web hosting/payload delivery), OpenSSH (encrypted remote access/surveillance), MySQL/MariaDB (target backend databases), and PostgreSQL (Metasploit data storage).

---

## Linux Service Management Syntax

While GUI options exist in Linux distributions like Kali, CLI management is required for security operations.

### Service Control Command Syntax

```bash
service <service_name> start|stop|restart

```

| Management Action | Command Example | Operational Purpose |
| --- | --- | --- |
| **Start Service** | `service apache2 start` | Launches service process in the background |
| **Stop Service** | `service apache2 stop` | Halts service execution immediately |
| **Restart Service** | `service apache2 restart` | Reloads daemon to apply plaintext configuration changes |

---

## Apache Web Server (HTTP)

Apache powers over 55% of global web servers. In penetration testing, Apache is used to set up phishing pages, deliver malware via Cross-Site Scripting (XSS), or host cloned sites paired with DNS redirection.

### LAMP vs. WAMP Stack

* **LAMP:** Linux, Apache, MySQL, PHP/Python (Standard Linux web deployment stack).
* **WAMP:** Windows, Apache, MySQL, PHP/Python (Windows variant).

### Installation & Web Directory Management

```bash
# Install Apache on Debian/Kali systems
apt-get install apache2

# Start Apache HTTP server
service apache2 start

```

* **Default Local URL:** `http://localhost/`
* **Default Root Directory:** `/var/www/html/`
* **Primary Web File:** `/var/www/html/index.html`

### Customizing Web Content

Replacing `/var/www/html/index.html` changes the served site:

```html
<html>
<body>
<h1>Hackers-Arise Is the Best! </h1>
<p> If you want to learn hacking, Hackers-Arise.com </p>
<p> is the best place to learn hacking!</p>
</body>
</html>

```

---

## OpenSSH & The Raspberry Spy Pi

SSH (Secure Shell) replaces insecure legacy tools like Telnet. It provides encrypted terminal access, user authentication, and access control lists.

### OpenSSH Startup

```bash
# Start SSH daemon on local system
service ssh start

```

### Building a Remote Surveillance System (Raspberry Spy Pi)

| Hardware / OS Parameter | Specification |
| --- | --- |
| **Hardware** | Raspberry Pi (v3 or Zero with built-in Wi-Fi) + Camera Module |
| **Operating System** | Raspbian OS |
| **Default User Credentials** | `pi` : `raspberry` |

#### Setup & Remote Connection Sequence

1. **Enable SSH on Pi GUI:** Preferences $\rightarrow$ Raspberry Pi Configuration $\rightarrow$ Interfaces $\rightarrow$ Enable SSH.
2. **Start SSH Service on Pi:** `service ssh start`
3. **Connect Camera:** Attach camera ribbon to the camera port (avoid contact with GPIO pins to prevent shorting).
4. **Identify IP Address:** Run `ifconfig` on Pi (e.g., `192.168.1.101`).
5. **Remote SSH Access from Kali:**
```bash
ssh pi@192.168.1.101

```


6. **Enable Camera Module via CLI Config:**
```bash
sudo raspi-config
# Select '6 Enable Camera' -> 'Finish' -> 'Reboot'

```



#### Executing Remote Surveillance Captures

```bash
# Take a photo and save to JPEG using raspistill utility
raspistill -v -o firstpicture.jpg

```

* `-v`: Enables verbose operational output.
* `-o`: Specifies destination output filename.

---

## MySQL / MariaDB Database Exploitation

MySQL (and its open-source fork MariaDB) powers backend databases for major platforms including WordPress, Facebook, LinkedIn, Twitter, Joomla, and Drupal.

### Corporate History

Developed by MySQL AB (1995) $\rightarrow$ Purchased by Sun Microsystems (2008) $\rightarrow$ Acquired by Oracle (2009). **MariaDB** was created as a community fork to ensure a fully open-source database system.

### Starting and Accessing MySQL

```bash
# Start MySQL/MariaDB service
service mysql start

# Local login with blank default password
mysql -u root -p

# Remote database authentication attempt
mysql -u root -p 192.168.1.101

```

> **Security Vulnerability:** Default MySQL/MariaDB installations have no root password set. Operating system credentials and database credentials are entirely separate.

### Key SQL Manipulation Commands

* `SELECT`: Retrieves data from specified tables/columns.
* `UNION`: Combines results from multiple `SELECT` statements.
* `INSERT`: Appends new records into a table.
* `UPDATE`: Modifies existing dataset records.
* `DELETE`: Erases records from a database.

### SQL Database Enumeration & Query Execution

```sql
-- All commands MUST terminate with a semicolon (;) or \g

-- View database users, host permissions, and passwords
select user, host, password from mysql.user;

-- Set root user password to 'hackers-arise' inside the mysql DB
use mysql;
update user set password = PASSWORD("hackers-arise") where user = 'root';

-- List all databases on target system
show databases;

-- Connect to a target database
use creditcardnumbers;

-- List all tables inside active database
show tables;

-- Display column schema, data types, keys, and defaults
describe cardnumbers;

-- Dump all table contents using the wildcard (*)
SELECT * FROM cardnumbers;

```

---

## PostgreSQL with Metasploit

PostgreSQL (Postgres) is an enterprise open-source relational database maintained by the PostgreSQL Global Development Group. It serves as the primary backend storage engine for the Metasploit Framework.

### Installation & Service Control

```bash
# Install PostgreSQL package
apt-get install postgres

# Start PostgreSQL service
service postgresql start

```

### Integrating PostgreSQL with Metasploit (`msfconsole`)

#### Step 1: Initialize Database & Switch User Context

```bash
# Launch Metasploit Framework console
msfconsole

# Initialize Metasploit database files (creates 'msf' & 'msf_test')
msf > msfdb init

# Switch to postgres system user context
msf > su postgres

```

#### Step 2: User Creation and Database Provisioning

```bash
# Create new database user role with password prompt
postgres@kali:/root$ createuser msf_user -P

# Create dedicated database owned by msf_user
postgres@kali:/root$ createdb --owner=msf_user hackers_arise_db

# Return to msfconsole prompt
postgres@kali:/root$ exit

```

#### Step 3: Metasploit Database Connection & Verification

```bash
# Connect msfconsole to PostgreSQL instance (User:Pass@Host/DB)
msf > db_connect msf_user:password@127.0.0.1/hackers_arise_db

# Confirm active database connectivity
msf > db_status

```

---

## Key Tips

* **Command Termination in MySQL** — Every SQL statement must explicitly end with a semicolon (`;`) or `\g` to execute; omitted semicolons result in prompt continuation without processing.
* **Wildcard Extraction (`*`)** — Using `SELECT * FROM <table>;` extracts every row and column simultaneously without typing field names manually.
* **Metasploit Acceleration** — Connecting Metasploit to PostgreSQL dramatically increases search speeds for modules and automatically logs scan findings and exploit results.
* **Camera Safety on Pi** — Never touch the Raspberry Pi camera module to the General Purpose Input/Output (GPIO) pins to avoid short-circuiting the board.

---

## Quick Command Chains

```bash
# Apache Delivery Setup Chain
service apache2 start && leafpad /var/www/html/index.html

# Full Metasploit Database Initialization Sequence
service postgresql start && msfconsole -x "msfdb init; exit"

# Remote Spy Pi Image Dump Command
ssh pi@192.168.1.101 "raspistill -v -o capture.jpg"

```

---

## Quick Start

Service Syntax (`service <name> start|stop|restart`) $\rightarrow$ Apache (`/var/www/html/index.html`) $\rightarrow$ SSH Surveillance (`ssh pi@IP` $\rightarrow$ `raspistill`) $\rightarrow$ MySQL Enumeration (`show databases;` $\rightarrow$ `SELECT *`) $\rightarrow$ Postgres Setup (`msfdb init` $\rightarrow$ `db_connect`)
