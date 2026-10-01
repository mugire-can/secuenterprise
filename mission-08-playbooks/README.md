# Mission 08 — Incident Response Playbooks

## Objective

Define realistic incident scenarios and document containment, eradication, and recovery procedures following the NIST SP 800-61 incident response lifecycle (Identification → Containment → Eradication → Recovery → Lessons Learned).

Three playbooks are documented: two fictional scenarios in keeping with the Enterprise-D theme, and one based on a **real incident** encountered during Mission 07.

---

## Playbook 01 — "Borg Intrusion" (Unauthorized Access / Lateral Movement)

### Scenario
An unknown entity gains unauthorized access to a bridge system and attempts to move laterally across the fleet's infrastructure, probing for weaknesses and escalating privileges.

### Identification
- Zabbix fires a trigger for abnormal CPU/memory utilization on a host outside of expected maintenance windows.
- Kibana log search (`program: "sshd"` or `program: "sudo"`) reveals repeated failed authentication attempts from an unrecognized source, followed by a successful login.
- Wireshark capture shows unexpected outbound connections from the affected host to unfamiliar internal addresses (lateral movement indicator).

### Containment
1. Isolate the affected host from the network (disable its network interface or apply a firewall rule blocking all non-essential traffic).
2. Disable the compromised user account.
3. Preserve the current system state (memory dump, running process list) for later analysis — do not reboot.

### Eradication
1. Identify and remove any unauthorized SSH keys, cron jobs, or startup scripts added by the intruder.
2. Patch the vulnerability that enabled initial access (e.g., weak password, unpatched service).
3. Rotate all credentials that may have been exposed.

### Recovery
1. Restore the host from a known-good backup/snapshot if integrity cannot be fully guaranteed.
2. Re-enable network access gradually, monitoring Zabbix/Kibana closely for recurrence.
3. Confirm normal service behavior for at least 24 hours before closing the incident.

### Lessons Learned
- Review and tighten SSH/sudo access policies fleet-wide.
- Add a Zabbix trigger specifically for repeated failed authentication attempts (brute-force detection).

---

## Playbook 02 — "Klingon Sabotage" (Denial of Service)

### Scenario
A hostile actor attempts to disrupt fleet operations by overwhelming a critical monitoring service with traffic, aiming to blind the crew to other, simultaneous threats.

### Identification
- Grafana dashboard shows a sharp, sustained spike in network/CPU metrics on a critical service (e.g., Zabbix server).
- The affected service stops responding to health checks; Prometheus marks its scrape target as `DOWN`.
- Elasticsearch Watcher fires an alert for abnormal log volume on the affected system.

### Containment
1. Rate-limit or temporarily block traffic from the identified source(s) at the firewall/network level.
2. Fail over monitoring responsibilities to a secondary system if available, to avoid losing visibility during the incident.

### Eradication
1. Confirm the attack traffic has stopped; identify and block the source IP range(s) permanently if malicious intent is confirmed.
2. Review service configuration for any exploitable resource-exhaustion weaknesses (e.g., unbounded connection limits).

### Recovery
1. Restart the affected service and confirm it returns to a healthy state in Grafana/Prometheus.
2. Validate that no data was lost during the outage window by cross-checking Zabbix/Elasticsearch continuity.

### Lessons Learned
- Implement rate limiting and connection caps on all externally-reachable monitoring endpoints.
- Add redundancy for critical monitoring services to avoid single points of failure.

---

## Playbook 03 — "Log Storm Cascade Failure" (Real Incident — Mission 07)

### Scenario
This is not fictional. During Mission 07, a misconfigured Logstash output (`stdout { codec => rubydebug }`) caused Logstash to write its own debug output back into `/var/log/syslog`, which it was simultaneously configured to ingest as an input — creating a self-reinforcing feedback loop. This generated tens of millions of log documents per day, exhausted the WSL virtual disk, and ultimately crashed the WSL service itself (`Wsl/Service/E_UNEXPECTED`), destabilizing the entire host machine.

### Identification
- Kibana queries for expected log sources (`program: "sudo"`) returned zero results — a signal that the pipeline was not behaving as expected.
- `df -h` revealed near-total disk exhaustion on the underlying storage.
- Windows host-level symptoms: File Explorer, browser, and other applications began freezing and throwing errors — a disk I/O-starvation cascade reaching beyond the VM into the host OS.
- `Get-PSDrive` confirmed the WSL virtual disk drive had only 0.02 GB free.

### Containment
1. Freed emergency disk space by deleting a large, no-longer-needed backup file (`ubuntu-backup.tar`, 23 GB).
2. Shut down and restarted the WSL instance (`wsl --shutdown`) to clear the crashed state.
3. Stopped the Logstash service to halt further log generation while investigating.

### Eradication
1. Root-caused the issue by inspecting a sample polluted document: the `message` field contained Logstash's own `rubydebug` output, confirming the feedback loop.
2. Removed the `stdout { codec => rubydebug }` line from the Logstash pipeline's `output` block — the fix that broke the loop at its source.
3. Added a defensive filter (`if [program] == "logstash" { drop {} }`) so that even if the loop were ever reintroduced, Logstash-origin events would never be indexed.
4. Deleted the polluted Elasticsearch indices and truncated the polluted `/var/log/syslog` file to clear the accumulated backlog.

### Recovery
1. Rewrote the Logstash pipeline configuration cleanly (using `tee <<'EOF'` to avoid editor line-wrap corruption, a secondary issue discovered during the fix).
2. Restarted Logstash and confirmed log volume grew at a normal, expected rate (not millions of documents per run).
3. Verified end-to-end: `program: "sudo"` queries in Kibana returned correct, parsed results.

### Lessons Learned
- **Never pipe a service's own debug/stdout output into a log source that the same service is configured to ingest** — this is a textbook feedback-loop risk.
- Disk usage should be actively monitored (a Zabbix trigger on disk utilization, which exists per Mission 01's template, would have caught this far earlier — a gap identified for future improvement).
- WSL2's dynamically-growing virtual disk can silently consume the entire host drive; this is a platform-specific risk worth documenting for anyone running this lab the same way.
- Editing multi-line config strings in `nano` can silently introduce line-wrap corruption; writing config files via heredoc (`tee <<'EOF'`) is safer for complex patterns.

---

## Summary

| Playbook | Type | Primary Detection Source | Key Takeaway |
|---|---|---|---|
| Borg Intrusion | Fictional | Zabbix + Kibana + Wireshark | Multi-tool correlation detects lateral movement |
| Klingon Sabotage | Fictional | Grafana + Prometheus + Watcher | Redundancy prevents monitoring blackout during DoS |
| Log Storm Cascade Failure | **Real** | Kibana (absence of expected data) + `df -h` | Feedback loops are a real and severe self-inflicted risk |

## Screenshots

See [`/docs/screenshots/mission-08/`](../docs/screenshots/mission-08/) for supporting evidence where applicable.
