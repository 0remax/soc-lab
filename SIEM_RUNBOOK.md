# Local SIEM Deployment & Detection Runbook

## 1. Infrastructure Architecture (`docker-compose.yml`)
To prevent handshake timeouts and enable the native alerting engine, ensure the following core configurations are present:

* **Elasticsearch Heap Control:** `ES_JAVA_OPTS=-Xms1g -Xmx1g` to stabilize low-memory environments.
* **X-Pack Security Status:** `xpack.security.enabled=false` to utilize plain-text HTTP communication on port `9200`.
* **Kibana Saved Objects Encryption:** The following absolute key must be set to securely store background alerting rules:
  ```yaml
  xpack.encryptedSavedObjects.encryptionKey: "a_minimum_32_character_unique_string_here"
  ```
## 2. Data Pipeline ingestion
  
1. **Agent:** Filebeat scraping `/var/log/auth.log`.
2. **Kibana Interface:** Create a Data View pointing explicitly to the pattern `filebeat-*` with `@timestamp` as the primary index marker field.
## 3. Detection Rule Blueprint
* **Rule Type:** `Elasticsearch Query`
* **Scope/Visibility:** `Logs`
* **Query String (KQL):** `message: "failed" or message: "failure"`
* **Grouping:** `GROUPED OVER top 1 'agent.hostname'`
* **Threshold Trigger:** `Count IS ABOVE 2` within a rolling `5 minutes` window.
## 🎯 4. The Core Lesson (**ESSENCE**)
The essence here is **Engineering Maturity**. True platform engineering doesn't stop when the dashboard turns green; it finishes when the build path is documented and repeatable. 
Save this markdown file alongside your `docker-compose.yml` file. You now have a complete, production-ready, locally verified SIEM playground. 
Is there any other security log source you want to onboard next, or would you like to explore adjusting this rule to forward alerts to an external webhook or email endpoint?
