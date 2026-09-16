# 🖖 SecuEnterprise — USS Enterprise-D Security Operations Center

> Detecting, monitoring, and responding to anomalous activity aboard the USS Enterprise-D — a hands-on Blue Team / SOC engineering project built on open-source monitoring, log analysis, and incident response tooling.

![Status](https://img.shields.io/badge/status-in%20progress-yellow)
![License](https://img.shields.io/badge/license-MIT-blue)
![Stack](https://img.shields.io/badge/stack-Zabbix%20%7C%20Prometheus%20%7C%20Grafana%20%7C%20ELK-informational)

---

## 📖 Overview

The USS Enterprise-D has detected abnormal activity across its systems. This project simulates a full **Security Operations Center (SOC) workflow**: from infrastructure monitoring and automated alerting, to threat analysis and incident response — all deployed on a virtualized lab environment.

The mission is broken down into 9 progressive stages, each building toward a complete monitoring, detection, and response pipeline.

## 🎯 Objectives

- Configure infrastructure monitoring systems for the Enterprise-D fleet
- Automate alerting for rapid anomaly detection
- Perform technical threat analysis on collected data
- Design and execute an incident response plan

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| **Monitoring** | Zabbix, Prometheus, Grafana |
| **Alerting & Log Pipeline** | Nagios, ELK Stack (Elasticsearch, Logstash, Kibana) |
| **Traffic / Log Analysis** | Wireshark, Kibana |
| **Incident Response** | Security Playbooks, TheHive, MISP |

## 🗺️ Mission Roadmap

| # | Mission | Status |
|---|---|---|
| 01 | Zabbix installation & host monitoring (CPU/RAM/Disk) | ✅ |
| 02 | Prometheus installation & scrape jobs | ✅ |
| 03 | Grafana installation & Prometheus dashboards | ✅ |
| 04 | Zabbix alert triggers & email notifications | ✅ |
| 05 | ELK Stack: Logstash pipelines, Kibana dashboards, Elasticsearch Watcher | ⬜ |
| 06 | Wireshark packet capture & suspicious traffic analysis | ⬜ |
| 07 | Log analysis & trend visualization with Kibana | ⬜ |
| 08 | Incident response playbooks (Borg intrusion, Klingon sabotage scenarios) | ⬜ |
| 09 | TheHive + MISP installation & IoC sharing | ⬜ |

*(Checkboxes will be updated to ✅ as each mission is completed and validated.)*

## 📂 Project Structure

```
SecuEnterprise/
├── mission-01-zabbix/
├── mission-02-prometheus/
├── mission-03-grafana/
├── mission-04-zabbix-alerts/
├── mission-05-elk-stack/
├── mission-06-wireshark/
├── mission-07-kibana-analysis/
├── mission-08-playbooks/
├── mission-09-thehive-misp/
├── docs/
│   └── screenshots/
├── .gitignore
├── LICENSE
└── README.md
```

Each `mission-XX/` folder contains its own `README.md` with setup steps, configuration files, and screenshots/proof of work for that stage.

## ⚙️ Prerequisites

- VirtualBox or VMware with an Ubuntu/Debian VM (2 vCPU / 2GB RAM / 20GB disk minimum)
- Basic Linux command line familiarity
- SSH access to the lab VM

## 🚀 Getting Started

Clone the repository:

```bash
git clone https://github.com/mugire-can/secuenterprise
cd secuenterprise
```

Follow the mission folders in order — each one documents the exact steps taken, commands used, and validation performed.

## 🧠 Skills Demonstrated

- Administering and securing virtualized infrastructure
- Detecting and handling security incidents
- Measuring and analyzing infrastructure security posture
- Implementing and optimizing infrastructure supervision

## 📚 Knowledge Base

- [Zabbix Documentation](https://www.zabbix.com/documentation)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Elasticsearch Documentation](https://www.elastic.co/guide/)
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)
- [Kibana Documentation](https://www.elastic.co/guide/en/kibana/current/index.html)
- [Elasticsearch Watcher](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-alerting.html)
- [Wireshark Documentation](https://www.wireshark.org/docs/)
- [TheHive Documentation](https://docs.strangebee.com/)
- [MISP Documentation](https://www.misp-project.org/documentation/)

## 📄 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

## 👤 Author

Built as part of a cybersecurity / infrastructure supervision training program (La Plateforme).

---

*Live long and monitor. 🖖*
