# Mission 05 — ELK Stack: Logstash, Kibana & Elasticsearch Watcher

## Objective

Install the full ELK Stack (Elasticsearch, Logstash, Kibana), ingest system logs, build visualization dashboards, and configure Elasticsearch Watcher to raise alerts on log volume anomalies.

## Environment

- **OS:** Ubuntu 24.04.5 LTS (WSL2, migrated from C: to D: drive mid-mission — see troubleshooting)
- **Elastic Stack version:** 8.19.21

## Steps Performed

### 1. Installed Elasticsearch

Added the official Elastic APT repository and installed Elasticsearch. Captured the auto-generated `elastic` superuser password and enrollment token from the install output (later reset via `elasticsearch-reset-password` after losing the original).

### 2. Installed and configured Logstash

Created a custom pipeline (`/etc/logstash/conf.d/enterprise-d-syslog.conf`) that:
- **Input:** tails `/var/log/syslog`
- **Filter:** parses lines with a `grok` pattern matching standard syslog format
- **Output:** ships parsed events to Elasticsearch under a daily-rotating index (`enterprise-d-logs-YYYY.MM.dd`)

### 3. Installed Kibana

Initial `apt install kibana` attempts repeatedly stalled during package unpacking (see troubleshooting). Kibana was ultimately installed **manually** from the official `.tar.gz` distribution, running as a dedicated systemd service pointing at `/usr/share/kibana-manual`.

### 4. Connected Kibana to Elasticsearch

Generated an enrollment token via `elasticsearch-create-enrollment-token --scope kibana`, completed the verification-code flow via `kibana-verification-code`, and logged in with the `elastic` superuser account.

### 5. Built visualizations

- **Discover:** created a data view for `enterprise-d-logs-*` (timestamp field `@timestamp`) and confirmed live syslog ingestion.
- **Dashboard** ("Enterprise-D Log Overview"): log volume over time (line chart) and log distribution by source program (pie chart).

### 6. Configured Elasticsearch Watcher

Created a scheduled watch (`enterprise_d_log_spike`) that queries log volume every minute and logs an alert if event count exceeds a threshold in the last minute — simulating detection of a traffic/log flood.

## Troubleshooting Log

| Issue | Cause | Fix |
|---|---|---|
| `Path "/var/lib/logstash/queue" must be a writable directory` — Logstash crash-looping | Ownership of `/var/lib/logstash` was incorrect after install | `sudo chown -R logstash:logstash /var/lib/logstash /var/log/logstash` |
| No documents ingested into Elasticsearch despite Logstash running | `/var/log/syslog` was empty — `rsyslog` is not installed/active by default on WSL2 | `sudo apt install rsyslog -y && sudo systemctl enable --now rsyslog` |
| `kibana` package install: `Input/output error` during unpack | The WSL2 virtual disk (`ext4.vhdx`) could not grow because the underlying Windows `C:` drive had only ~1.5 GB free | Freed space and, more durably, migrated the entire WSL distro from `C:` to a secondary drive via `wsl --export` / `wsl --unregister` / `wsl --import` |
| `apt install kibana` hung indefinitely at 20% unpack, `dpkg` stuck in uninterruptible sleep (`D` state) | Combination of low available RAM (Elasticsearch + Logstash already consuming most of it) and, primarily, **Windows Defender real-time scanning** every one of the thousands of small files inside Kibana's `node_modules` as they were written to the WSL virtual disk | Added Windows Defender exclusions for the `vmmem`/`wsl.exe` processes and the WSL distro path; when the `apt`/`dpkg` path remained impractically slow, installed Kibana manually from the official `.tar.gz` archive instead, bypassing `dpkg` entirely |
| Kibana setup asked for a "verification code" instead of accepting the `elastic` user directly | Expected behavior — the enrollment screen authenticates the `kibana_system` service account, not `elastic` | Ran `kibana-verification-code` on the server, entered the 6-digit code, then logged into the actual Kibana UI with the `elastic` superuser |
| `security_exception: current license is non-compliant for [watcher]` | Elasticsearch defaults to a Basic license, which does not include Watcher (part of the X-Pack "platinum" feature set) | Activated the 30-day trial license: `POST _license/start_trial?acknowledge=true` |

## Result

A fully functional log pipeline is in place: system logs flow from `/var/log/syslog` through Logstash into Elasticsearch, are visualized in Kibana dashboards, and are monitored by an Elasticsearch Watcher that flags abnormal log volume — the foundation for the deeper log analysis performed in Mission 07.

## Screenshots

See [`/docs/screenshots/mission-05/`](../docs/screenshots/mission-05/) for the Discover view, the dashboard.
