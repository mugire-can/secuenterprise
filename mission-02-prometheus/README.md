# Mission 02 — Prometheus Installation & Scrape Jobs

## Objective

Install Prometheus and Node Exporter to collect system-level metrics (CPU, memory, disk, network) from the Enterprise-D bridge system, and confirm scrape targets are healthy.

## Environment

- **OS:** Ubuntu 24.04.5 LTS (WSL2)
- **Prometheus version:** 2.55.1
- **Node Exporter version:** 1.8.2

## Steps Performed

### 1. Create a dedicated system user and directories

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

### 2. Download and install Prometheus binaries

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.55.1/prometheus-2.55.1.linux-amd64.tar.gz
tar xvf prometheus-2.55.1.linux-amd64.tar.gz
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

### 3. Create the systemd service

`/etc/systemd/system/prometheus.service` — runs Prometheus as the dedicated `prometheus` user, pointing at `/etc/prometheus/prometheus.yml` and storing time-series data under `/var/lib/prometheus/`.

### 4. Install Node Exporter

Downloaded, installed to `/usr/local/bin/node_exporter`, and registered as a systemd service (`node_exporter.service`) running under a dedicated unprivileged user. Node Exporter exposes host metrics on port `9100`.

### 5. Configure scrape jobs

Added to `/etc/prometheus/prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'enterprise-d-bridge-node'
    static_configs:
      - targets: ['localhost:9100']
```

### 6. Start services

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
sudo systemctl enable --now prometheus
```

## Verification

- `sudo systemctl status prometheus` / `node_exporter` → both `active (running)`
- `http://<host-ip>:9090/targets` → both `prometheus` and `enterprise-d-bridge-node` jobs reporting **State: UP**

## Result

Prometheus is actively scraping both its own internal metrics and full host-level system metrics (CPU, memory, disk, filesystem, network) from Node Exporter every scrape interval, ready to be visualized in Grafana (Mission 03).

## Screenshots

See [`/docs/screenshots/mission-02/`](../docs/screenshots/mission-02/) for the Prometheus targets page showing both jobs UP.
