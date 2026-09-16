# Mission 03 — Grafana Installation & Prometheus Integration

## Objective

Install Grafana, connect it to the existing Prometheus instance as a data source, and build a dashboard visualizing live host metrics collected via Node Exporter.

## Environment

- **OS:** Ubuntu 24.04.5 LTS (WSL2)
- **Grafana version:** latest (official APT repo, OSS edition)
- **Data source:** Prometheus (local, `localhost:9090`)

## Steps Performed

### 1. Add the official Grafana APT repository

```bash
sudo apt install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

### 2. Install and start Grafana

```bash
sudo apt update
sudo apt install grafana -y
sudo systemctl enable --now grafana-server
```

### 3. Initial login & password change

Logged in at `http://<host-ip>:3000` with default credentials (`admin`/`admin`) and set a new admin password on first login.

### 4. Add Prometheus as a data source

**Connections → Data sources → Add data source → Prometheus**
- URL: `http://localhost:9090`
- Verified with **Save & test** — connection successful.

### 5. Build the monitoring dashboard

Imported the community **Node Exporter Full** dashboard (ID `1860`), pointed at the Prometheus data source — provides pre-built panels for CPU, memory, disk I/O, network, and filesystem usage with no manual query writing required.

## Verification

- Dashboard renders live, auto-refreshing panels for CPU utilization, memory usage, and disk space, sourced directly from Prometheus/Node Exporter metrics collected in Mission 02.

## Result

Enterprise-D's bridge systems are now visually monitored in real time through a centralized Grafana dashboard, completing the observability stack (Zabbix + Prometheus + Grafana).

## Screenshots

See [`/docs/screenshots/mission-03/`](../docs/screenshots/mission-03/) for the Grafana dashboard showing live metrics.
