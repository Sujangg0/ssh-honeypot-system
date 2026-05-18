# SSH Honeypot — Cowrie + ELK Stack

A dockerized SSH honeypot that captures attacker activity and visualizes it on a Kibana dashboard with a live GeoIP world map.

## Blog Post

*Coming soon*

## Stack

- **Cowrie** — fake SSH server on port 2222
- **Elasticsearch** — stores and indexes the logs
- **Filebeat** — ships Cowrie logs to Elasticsearch
- **Kibana** — dashboard and GeoIP visualization

## Requirements

- Linux (Ubuntu 22.04 / 24.04 recommended)
- Docker + Docker Compose
- At least 4GB RAM

## Setup

**1. Clone the repo**
```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
cd YOUR_REPO_NAME
```

**2. Create directories and fix permissions**
```bash
mkdir -p cowrie/log elasticsearch/elasticsearch-data
sudo chmod -R 777 cowrie elasticsearch
```

**3. Fix Elasticsearch kernel requirement**
```bash
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

**4. Start the stack**
```bash
docker compose up -d
```

**5. Create the GeoIP pipeline**
```bash
curl -X PUT "localhost:9200/_ingest/pipeline/cowrie-geoip-pipeline" \
-H "Content-Type: application/json" \
-d '{
  "description": "GeoIP pipeline for Cowrie logs",
  "processors": [{ "geoip": { "field": "src_ip", "target_field": "source.geo" } }]
}'
```

**6. Access Kibana**

Use an SSH tunnel if on a remote server:
```bash
ssh -L 5601:localhost:5601 user@YOUR_SERVER_IP
```
Then open `http://localhost:5601` in your browser.

**7. Import the dashboard**

In Kibana go to **Stack Management → Saved Objects → Import** and upload `cowrie-dashboard.ndjson`.

## Port Reference

| Port | Service       | Expose Publicly |
|------|---------------|-----------------|
| 2222 | Cowrie (SSH)  | Yes             |
| 9200 | Elasticsearch | No              |
| 5601 | Kibana        | No              |

