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
```
The Orchestrated Sequencing Solution
To prevent resource exhaustion (NS_ERROR_NET_EMPTY_RESPONSE) caused by parallel JVM and Node.js compilation spikes, the infrastructure must be initialized using staged dependency boot ordering:
1. Initialize the storage engine exclusively:
   ```bash
   sudo docker compose up -d elasticsearch
   ```
2. Wait for the REST API endpoint handshake loop to clear:
   ```bash
   curl http://localhost:9200
   ```
3. Boot the visualization dashboard and telemetry collector once the database declares healthy status:
   ```bash
   sudo docker compose up -d kibana
   sudo docker start soc-filebeat-agent
   ```
## 🎯 Forensic Capabilities & Verified Simulations (ESSENCE)
The true value of this deployment was validated by executing a credential attack simulation and auditing the resulting log generation cycle.
### Verified Incident: Host-Based Brute Force Simulation
An interactive brute-force session was executed by driving multiple high-privilege execution anomalies against the local security system boundary (`sudo ls /root`).
* **System Response:** The Linux Pluggable Authentication Module (PAM) tripped its lockout boundary rule, logging `sudo: maximum 3 incorrect authentication attempts` directly to `/var/log/auth.log`.
* **SIEM Capture:** Filebeat intercepted the delta modification instantly, passing the raw string to Elasticsearch.
* **Analyst Viewpoint:** Utilizing a custom Kibana Data View mapped against `filebeat-*` and filtered by the columns `log.file.path` and `message`, the attack pattern was successfully tracked and contextualized via the following KQL query:
```plaintext
message: "failure" or message: "failed"
```

##🛠️ Repository File Structure
* `docker-compose.yml` - Contains multi-container definitions, shared host networking arrays, and internal resource constraints.
* `config/filebeat.yml` - Explicit path harvesting configurations pointing to system event brokers with disabled Elasticsearch security overrides for localized host testing.
## 🚨 SIEM Behavioral Detection Engineering

To transition the platform from a passive log repository into an active defense mechanism, a real-time behavioral alerting rule was engineered directly into the Kibana analytics engine.

### Rule Configuration: Host-Based Brute Force Threshold Detection
Rather than relying on manual monitoring, the SIEM dynamically parses incoming logs for brute-force telemetry spikes using the following structured threshold rule:

*   **Rule Type:** Threshold Condition Rule
*   **Target Data Index:** `filebeat-*`
*   **Behavioral Query Filter (KQL):** `message: "failed" or message: "failure"`
*   **Aggregation Vector:** Grouped by `agent.hostname`
*   **Condition Threshold:** Count `> 2` matching events within a `5-minute` rolling window.

### Operational Intent
By grouping the threshold condition explicitly by `agent.hostname`, the SIEM correlates localized noise. If an adversary attempts 50 rapid credential sprays against a single asset, the rule collapses the noise and fires a single, actionable security alert case rather than triggering 50 individual event notifications—effectively preventing analyst alert fatigue.
