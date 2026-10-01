# Mission 07 — Log Analysis with Kibana

## Objective

Search for specific security-relevant events (privilege escalation, authentication activity) in the ingested logs and build trend visualizations over time using Kibana.

## Environment

- **OS:** Ubuntu 24.04.5 LTS (WSL2)
- **Kibana version:** 8.19.21

## Steps Performed

### 1. Searched for specific events using KQL

- `program: "sudo"` — privilege escalation / `sudo` usage audit
- `program: "sshd"` — SSH authentication activity
- `log_message: *error* or log_message: *failed*` — general failure/error events

### 2. Extended the log pipeline to cover authentication logs

The original pipeline (Mission 05) only ingested `/var/log/syslog`, which does not contain `sudo`/authentication events on Debian/Ubuntu — those are written to `/var/log/auth.log`. Added a second `file` input for `/var/log/auth.log` with its own dedicated Logstash `sincedb` file.

### 3. Built a trend visualization

Added a **Lens** panel to the Mission 05 dashboard: a date-histogram of log volume over time, broken down by `program`, to visualize which services generate the most log activity and when — a baseline for spotting anomalies.

## Troubleshooting Log

This mission surfaced the most complex incident of the whole project — a genuine cascading failure that took the lab environment down and required root-cause analysis across several layers.

| Issue | Cause | Fix |
|---|---|---|
| `program: "sudo"` / `program: "sshd"` returned no results | Pipeline only ingested `syslog`, not `auth.log` where authentication events live | Added a second Logstash `file` input for `/var/log/auth.log` |
| **Catastrophic WSL failure** (`Wsl/Service/E_UNEXPECTED`), Windows host became unresponsive (Explorer, browser, media player all freezing) | The WSL virtual disk drive filled to 0.02 GB free. Root cause traced to a **Logstash feedback loop**: the `stdout { codec => rubydebug }` output was writing Logstash's own debug output back into `/var/log/syslog` (via systemd/journald), which Logstash itself was tailing as an input — creating an exponential self-ingestion loop that produced tens of millions of documents per day | Removed a large stale backup file to regain disk headroom, restarted WSL, then **removed the `stdout { codec => rubydebug }` line** from the Logstash output block to break the feedback loop; deleted the polluted indices |
| Grok filter failing on `auth.log` lines (`_grokparsefailure` tag, no `program`/`timestamp` fields extracted) | `auth.log` timestamps use full ISO8601 format, not the legacy `SYSLOGTIMESTAMP` pattern the grok filter expected | Updated the grok pattern to `%{TIMESTAMP_ISO8601:timestamp} %{SYSLOGHOST:hostname} %{DATA:program}...` |
| Both `syslog` and `auth.log` inputs shared the same `sincedb_path` | Copy-paste error in the pipeline config — two file inputs pointed at the same sincedb file, causing one to silently overwrite the other's read position | Gave each input its own dedicated sincedb file (`sincedb_syslog`, `sincedb_authlog`) |
| Grok pattern silently broken (line wrapped mid-token by the editor) | Editing the multi-line config in `nano` introduced an unintended line break inside `%{SYSLOGHOST:hostname}` | Rewrote the config file in one shot using `tee <<'EOF' ... EOF` to avoid editor line-wrap artifacts, then verified with `grep -n` that the pattern was on a single line |
| Disk filled again after re-enabling ingestion | `sincedb` reset combined with the still-polluted `/var/log/syslog` (megabytes of accumulated feedback-loop noise) caused a large backlog to be re-read on restart | `sudo truncate -s 0 /var/log/syslog` to clear the polluted backlog before restarting Logstash with the corrected pipeline |

## Result

After resolving a genuine cascading infrastructure incident (feedback loop → disk exhaustion → WSL crash → host instability), the log pipeline now correctly ingests and parses both `syslog` and `auth.log`, with `program: "sudo"` and similar queries returning accurate, correctly-parsed results in Kibana. This mission doubled as a real-world incident response exercise, directly relevant to Mission 08.

## Screenshots

See [`/docs/screenshots/mission-07/`](../docs/screenshots/mission-07/) for the KQL search results and the log volume trend visualization.
