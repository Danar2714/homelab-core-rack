# Monitoring Server Raspberry Pi 3 B+ (DietPi)

## A. Tailscale Configuration

### 1. Using Tailscale for remote access

This Raspberry Pi runs DietPi and acts as the monitoring server for the homelab.  
Tailscale is installed so that any web interface or host inside the lab network can be reached securely over the Tailscale VPN from outside the house.

---

#### 1.1 Installing the Tailscale service

Tailscale is installed from the official script and the service is enabled so it starts automatically on boot.

```bash
curl -fsSL https://tailscale.com/install.sh | sh

sudo systemctl enable tailscaled
sudo systemctl start tailscaled
sudo systemctl status tailscaled
```

- The `install.sh` script adds the Tailscale repository and installs `tailscaled`.
- `systemctl` is used to enable, start and verify the Tailscale daemon.

---

#### 1.2 Enabling IP forwarding on DietPi

Before advertising the LAN subnet through Tailscale, IP forwarding is enabled so the Raspberry Pi can route packets between the Tailscale interface and the physical LAN (`eth0`).

```bash
sudo tee /etc/sysctl.conf > /dev/null <<EOF
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
EOF

sudo sysctl -p
```

<img src="../docs/ip_foward.png" alt="Enabling IPv4 and IPv6 forwarding via sysctl on DietPi" width="50%" />

**Figure 1 – IP forwarding configuration**  
`sysctl.conf` is overwritten with forwarding enabled, and `sysctl -p` reloads the settings so the Raspberry Pi can forward traffic between Tailscale and the LAN.

---

#### 1.3 Bringing the node online and advertising the homelab subnet

With forwarding enabled, the node is brought online and configured to advertise the LAN subnet `172.16.0.0/24` so other Tailscale devices can reach machines in that network.

```bash
# First-time login in the browser:
sudo tailscale up

# Then, advertise the homelab subnet:
sudo tailscale up --advertise-routes=172.16.0.0/24 --accept-routes
```

<img src="../docs/announce_network.png" alt="tailscale up command advertising the 172.16.0.0/24 subnet" width="50%" />

**Figure 2 – Advertising the 172.16.0.0/24 subnet**  
The initial `tailscale up` command registers the DietPi node, and the second invocation advertises the `172.16.0.0/24` route with `--advertise-routes` and allows receiving routes from other nodes with `--accept-routes`.

---

#### 1.4 Approving the subnet route in the Tailscale admin console

The advertised route must be approved from the Tailscale admin console before it becomes active.

Steps:

1. Log in to the Tailscale admin panel.
2. Locate the device **dietpi**.
3. Open **Edit route settings**.
4. Check the subnet route `172.16.0.0/24` and save.

<img src="../docs/Edit_route_settings.png" alt="Approving the 172.16.0.0/24 subnet route for dietpi in the Tailscale admin console" width="50%" />

**Figure 3 – Approving the advertised subnet route**  
The panel shows that `172.16.0.0/24` is now approved, so other Tailscale clients can reach any host in that subnet via the DietPi router.

---

#### 1.5 Resulting remote access

With Tailscale running, IP forwarding enabled, and the subnet route approved, any device connected to Tailscale can now access homelab services (such as Proxmox, router web GUIs, and monitoring dashboards) by using their LAN addresses in the `172.16.0.0/24` network.

---

## B. Zabbix Server Configuration

### 1. Installing the Zabbix repository and packages

This section covers adding the official Zabbix repository on DietPi (Debian 13), updating the package lists and installing the Zabbix server, frontend and agent.

---

#### 1.1 Downloading the Zabbix repository package

The Zabbix repository `.deb` package is downloaded into `/root` using `wget`.

```bash
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_7.0-2+debian13_all.deb
```

<img src="../docs/Zabbix/rpi_install_zabbix_server1.png" alt="Downloading the Zabbix 7.0 repository package on DietPi" width="50%" />

**Figure 4 – Downloading the Zabbix repository package**  
The command downloads `zabbix-release_7.0-2+debian13_all.deb` and the `ls` output confirms the file is present in the current directory.

---

#### 1.2 Installing the Zabbix repository

The downloaded package is installed with `dpkg` so that APT can use the Zabbix repository.

```bash
dpkg -i zabbix-release_7.0-2+debian13_all.deb
```

<img src="../docs/Zabbix/rpi_install_zabbix_server2.png" alt="Installing the Zabbix repository package with dpkg" width="50%" />

**Figure 5 – Enabling the Zabbix APT repository**  
The system unpacks and sets up the `zabbix-release` package, which adds the Zabbix repository entries under `/etc/apt/sources.list.d/`.

---

#### 1.3 Updating APT indexes

After enabling the repository, the package lists are updated so the new Zabbix packages become available.

```bash
apt update
```

<img src="../docs/Zabbix/rpi_apt_update.png" alt="Running apt update after adding the Zabbix repository" width="50%" />

**Figure 6 – Updating package lists**  
The output shows the standard Debian repositories plus the new `repo.zabbix.com` entries for Zabbix 7.0 on Debian Trixie.

---

#### 1.4 Installing Zabbix server, frontend and agent

With the repository configured, the Zabbix components and MariaDB server are installed in a single command.

```bash
apt install -y \
  mariadb-server \
  zabbix-server-mysql \
  zabbix-frontend-php \
  zabbix-apache-conf \
  zabbix-sql-scripts \
  zabbix-agent2
```

<img src="../docs/Zabbix/rpi_install_zabbix_server3.png" alt="Installing Zabbix server, frontend and agent with apt" width="50%" />

**Figure 7 – Installing Zabbix and MariaDB packages**  
This installs the database server, Zabbix server daemon (MySQL/MariaDB backend), the PHP frontend with Apache integration, the SQL schema scripts and the Zabbix Agent 2.

---

### 2. Securing the MariaDB server

Before creating the Zabbix database, the MariaDB instance is hardened using the `mariadb-secure-installation` helper.

```bash
mariadb-secure-installation
```

<img src="../docs/Zabbix/rpi_mariadb1.png" alt="Running mariadb-secure-installation and setting the root password" width="50%" />

**Figure 8 – Initial MariaDB secure installation steps**  
During the wizard, a root password is set and Unix socket authentication options are confirmed according to the prompts.

<img src="../docs/Zabbix/rpi_mariadb2.png" alt="Removing anonymous users, test database and reloading privileges in MariaDB" width="50%" />

**Figure 9 – Removing test database and tightening access**  
Anonymous users are removed, remote root login is disabled, the default `test` database is dropped and the privilege tables are reloaded to apply the changes.

---

### 3. Creating the Zabbix database and user

Once MariaDB is secured, the database and user for Zabbix are created.  
First, the MariaDB shell is opened as root:

```bash
mariadb -u root -p
```

Inside the MariaDB prompt, the following SQL statements are executed:

```sql
CREATE DATABASE zabbix
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_bin;

CREATE USER 'zabbix'@'localhost'
  IDENTIFIED BY '';

GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';

FLUSH PRIVILEGES;
EXIT;
```

<img src="../docs/Zabbix/rpi_mariadb_db_creation.png" alt="Creating the Zabbix database and user in MariaDB" width="50%" />

**Figure 10 – Creating the Zabbix database and granting privileges**  
The database `zabbix` is created with UTF-8 settings, a local user `zabbix` is defined with an (initially) empty password, and full privileges on the `zabbix` schema are granted. In a real deployment, a strong password should be configured instead of an empty value.

---

### 4. Importing the initial Zabbix schema

Zabbix provides a pre-built schema and data file in compressed form.  
It is imported into the `zabbix` database using `zcat` piped into `mariadb`:

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz \
  | mariadb -u zabbix -p zabbix
```

<img src="../docs/Zabbix/rpi_zabbix_schema.png" alt="Importing the Zabbix schema into MariaDB using zcat and mariadb" width="50%" />

**Figure 11 – Importing the Zabbix SQL schema**  
The command expands `server.sql.gz` and feeds it directly to MariaDB, populating all required tables and initial data in the `zabbix` database.

---

### 5. Configuring the Zabbix server to use MariaDB

The Zabbix server configuration file is edited so the daemon knows how to connect to the MariaDB instance.

```bash
nano /etc/zabbix/zabbix_server.conf
```

<img src="../docs/Zabbix/rpi_zabbix_server_config1.png" alt="Opening the Zabbix server configuration file in nano" width="50%" />

**Figure 12 – Zabbix server configuration file**  
The file `/etc/zabbix/zabbix_server.conf` contains the main server parameters, including database connection options.

In the database section, the `DBUser` and `DBPassword` parameters are set to match the MariaDB user created earlier. For documentation purposes, the password field is left empty:

```text
DBUser=zabbix
DBPassword=
```

<img src="../docs/Zabbix/rpi_zabbix_server_config2.png" alt="Setting DBUser and empty DBPassword in zabbix_server.conf" width="50%" />

**Figure 13 – Database credentials in zabbix_server.conf**  
`DBUser` is already set to `zabbix` by default; only `DBPassword` needs to be uncommented. A real deployment should use a strong password here and in the corresponding MariaDB user configuration.

---


