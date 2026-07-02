# 🛡️ Self-Hosted SIEM Lab: Elastic Stack & Filebeat Ingestion Pipeline

A production-representative Security Operations Center (SOC) home lab environment designed to ingest, parse, and analyze system authentication telemetry in real time. This project deploys a lightweight Elastic Stack footprint inside an orchestrated, resource-constrained Ubuntu environment to simulate realistic telemetry ingestion and attack auditing.

---

## 🧬 Architectural Strategy (W-H-E Method)

### 🧠 1. Strategic Intent (WHY)
Modern security operations rely on rapid telemetry correlation to catch adversarial footprints before lateral movement occurs. This repository documents how to build a fully functional SIEM (Security Information and Event Management) framework capable of tracking operational system health and security boundaries under a tight 4GB RAM hardware envelope.

### 🧱 2. Structural Components (WHAT)
The environment orchestrates three distinct operational layers using Docker Containers:
*   **Database Engine:** Elasticsearch (`8.11.1`) acting as the high-throughput indexing node, bound with a strict 1GB JVM heap limit.
*   **Visualization Console:** Kibana (`8.11.1`) providing the web dashboard interface and KQL (Kibana Query Language) querying runtime.
*   **Log Shipper:** Filebeat Agent actively harvesting system telemetry footprints directly from `/var/log/auth.log` and `/var/log/syslog`.

### ⚙️ 3. Technical Mechanics (HOW)

```text
[ Host Machine (8GB RAM) ]
         │
         ▼  (Resource Split)
[ VirtualBox Ubuntu VM (4GB RAM / 4 vCPUs) ]
         │
         ├─► [ Container: Elasticsearch (Port 9200) ] ◄──┐ (JSON Indexing)
         │                                               │
         ├─► [ Container: Kibana (Port 5601) ]           │
         │                                               │
         └─► [ Container: Filebeat Agent ] ──────────────┘
                    │
                    ▼ (Live Harvesting)
             /var/log/auth.log  &  /var/log/syslog
