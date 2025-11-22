# Backup Server Intel NUC Celeron (Alpine)

## A. Zabbix Agent Configuration

### 1. Installing Zabbix Agent 2 on Alpine

This Intel NUC Celeron runs Alpine Linux and acts as the backup server in the homelab.  
To integrate it into the Zabbix monitoring stack, **Zabbix Agent 2** is installed and configured to report to the DietPi Zabbix server at `172.16.0.5`.

---

#### 1.1 Updating repositories and installing the agent

First, the Alpine package indexes are updated and the Zabbix Agent 2 packages are installed together with `logrotate` and the OpenRC service files.

```bash
apk update
apk add zabbix-agent2 zabbix-agent2-openrc logrotate
```

<img src="../docs/Zabbix/backupserver_install_zabbix_agent.png" width="50%" />

**Figure 1 – Installing Zabbix Agent 2 on Alpine**  
The command `apk add` pulls `zabbix-agent2`, its OpenRC integration and the required libraries, preparing the system to run the agent as a managed service.

---

#### 1.2 Creating the log directory for Zabbix

By default, the agent expects a writable log directory under `/var/log/zabbix`.  
On this Alpine installation the directory is created manually and ownership is given to the `zabbix` user:

```bash
mkdir -p /var/log/zabbix
chown zabbix:zabbix /var/log/zabbix
```

<img src="../docs/Zabbix/backupserver_logs_zabbix_agent.png" width="50%" />

**Figure 2 – Preparing the Zabbix log directory**  
This ensures that `zabbix-agent2` can create and rotate its own log files without permission issues.

---

### 2. Configuring Zabbix Agent 2

The main configuration file is `/etc/zabbix/zabbix_agent2.conf`.  
At minimum, the **Server**, **ServerActive** and **Hostname** parameters are adjusted so the agent can communicate with the central Zabbix server.

---

#### 2.1 Setting server and hostname parameters

```bash
nano /etc/zabbix/zabbix_agent2.conf
```

Relevant lines:

```text
Server=172.16.0.5
ServerActive=172.16.0.5
Hostname=Backup server
```

<img src="../docs/Zabbix/backupserver_config_zabbix_agent.png" width="80%" />

**Figure 3 – Zabbix agent configuration on the Backup Server**  
- `Server` lists the IPs allowed to query the agent (passive checks).  
- `ServerActive` defines where the agent should send active check data.  
- `Hostname` must match the host name configured later in the Zabbix web interface (e.g. *Backup Server*).

---

### 3. Enabling and starting the agent service

With the configuration in place, `zabbix-agent2` is enabled in OpenRC so it starts at boot, and then started immediately.

```bash
rc-update add zabbix-agent2
rc-service zabbix-agent2 start
rc-service zabbix-agent2 status
```

<img src="../docs/Zabbix/backupserver_start_zabbix_agent.png" width="60%" />

**Figure 4 – Starting and enabling Zabbix Agent 2 on Alpine**  
The `rc-update` command adds the service to the default runlevel, while `rc-service ... status` confirms that the agent is now running and ready to be discovered by the Zabbix server.

---
