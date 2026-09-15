# Mission 01 — Zabbix Installation & Host Monitoring

## Objective

Install and configure Zabbix Server on an Ubuntu 24.04 LTS host (running under WSL2), add the local system as a monitored host, and visualize CPU/RAM/Disk metrics.

## Environment

- **OS:** Ubuntu 24.04.5 LTS (WSL2)
- **Zabbix version:** 7.0 LTS
- **Database:** MySQL 8.0
- **Web server:** Apache2 + PHP

## Steps Performed

### 1. Add the Zabbix repository

```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-2+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_7.0-2+ubuntu24.04_all.deb
sudo apt update
```

### 2. Install Zabbix server, frontend, agent, and MySQL

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent mysql-server -y
```

### 3. Create the Zabbix database and user

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER zabbix@localhost IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;
SET GLOBAL log_bin_trust_function_creators = 1;
```

### 4. Import the initial schema

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

### 5. Configure the database password and start services

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

### 6. Complete the web setup wizard

`http://<host-ip>/zabbix/setup.php` → configure DB connection → login with default credentials (`Admin` / `zabbix`).

### 7. Add a monitored host

**Data collection → Hosts → Create host**
- Host name: `Enterprise-D-Bridge`
- Host group: `Enterprise-D-Bridge`
- Template: `Linux by Zabbix agent`
- Agent interface: `127.0.0.1:10050`

### 8. Verify data collection

**Monitoring → Latest data** — confirmed CPU, memory, and disk metrics updating in real time, and generated a graph from the collected data.

## Troubleshooting Log

Real issues encountered during this mission and how they were resolved — kept here for future reference.

| Issue | Cause | Fix |
|---|---|---|
| `wget` returned 404 for `zabbix-release` package | Zabbix changed their repo URL structure (dropped the `/release/` path segment) | Used `https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/...` instead of `.../7.0/release/ubuntu/pool/...` |
| `ERROR 1045: Access denied for user 'zabbix'@'localhost'` | User/password mismatch after initial creation | Reset via `ALTER USER 'zabbix'@'localhost' IDENTIFIED BY '...'` |
| `ERROR 2002: Can't connect to local MySQL server through socket` | MySQL service was not running | `sudo systemctl start mysql` |
| `mysql.service` failed — *"MySQL has been frozen to prevent damage to your system"* | Leftover/incompatible data in `/var/lib/mysql` from a prior partial install | Fully purged (`apt purge mysql-server`, removed `/var/lib/mysql`, `/etc/mysql`) and reinstalled clean |
| `ERROR 1050: Table 'role' already exists` on schema import | Schema was partially imported in a previous attempt | Dropped and recreated the `zabbix` database before re-importing |
| `zabbix_server.conf` was empty after install | Package `zabbix-server-mysql` was interrupted mid-install | `sudo apt install --reinstall zabbix-server-mysql -y` |
| `apt install --reinstall` hung at 91% | systemd was not actually active in WSL2 despite `systemd=true` in `/etc/wsl.conf` (config had not been applied yet — required a full WSL restart) | `wsl --shutdown` from PowerShell, then reopened WSL and confirmed `ps -p 1 -o comm=` returned `systemd` |
| Setup wizard warning: *"Locale for language en_US is not found"* | `en_US.UTF-8` locale not generated on the system | `sudo apt install locales`, `sudo locale-gen en_US.UTF-8`, `sudo update-locale LANG=en_US.UTF-8` |
| Login with DB credentials (`zabbix`/`<db password>`) failed on the web interface | Confused MySQL database credentials with the Zabbix frontend login | Frontend login uses separate default credentials: `Admin` / `zabbix` |
| *"Cannot add host — Field 'groups' is mandatory"* | No host group selected when creating the host | Created a new host group (`Enterprise-D-Bridge`) and assigned it |

## Result

Zabbix Server is fully operational, monitoring the local host's CPU, memory, and disk usage with live graphs available in the web dashboard.

## Screenshots

See [`/docs/screenshots/mission-01/`](../docs/screenshots/mission-01/) for the setup wizard, host configuration, and Latest Data graphs.
