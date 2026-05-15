### SNA_Project

## Folder Structure

```
monitoring-stack/
├── prometheus/
│   ├── prometheus.yml       # scrape config + alerting rules pointer
│   └── alerts.yml           # CPU, memory, disk, and instance-down alerts
├── alertmanager/
│   └── config.yml           # email (+ optional Slack) routing
├── loki/
│   └── loki-config.yaml     # log storage config
├── promtail/
│   └── promtail-config.yaml # log collection (syslog + Docker)
├── nginx/
│   └── nginx.conf           # reverse proxy in front of Grafana
├── docker-compose.yml
└── .github/
    └── workflows/
        └── deploy.yml       # validates configs then SSH-deploys
```

## Start

### 1. Prerequisites
- Docker + Docker Compose V2
- Linux host (or Docker Desktop on macOS/Windows)

### 2. Clone & configure

```bash
git clone https://github.com/YOUR_USER/monitoring-stack.git
cd monitoring-stack
```

Edit the files that contain placeholder values:

| File | What to change |
|------|---------------|
| `alertmanager/config.yml` | Your email address, Gmail App Password |
| `promtail/promtail-config.yaml` | `host: your-server` - your hostname |
| `docker-compose.yml` | `GF_SECURITY_ADMIN_PASSWORD` |

### 3. Start the stack

```bash
docker compose up -d
```

### 4. Verify all containers are running

```bash
docker compose ps
```

You should see 7 containers: `node-exporter`, `prometheus`, `alertmanager`, `loki`, `promtail`, `grafana`, `nginx`.

### 5. Access the services

| Service | URL | Credentials |
|---------|-----|-------------|
| Grafana (via Nginx) | http://localhost | admin / admin |
| Grafana (direct) | http://localhost:3000 | admin / admin |
| Prometheus | http://localhost:9090 | — |
| Alertmanager | http://localhost:9093 | — |
| Node Exporter | http://localhost:9100/metrics | — |
| Loki | http://localhost:3100 | — |

### 6. Configure Grafana data sources

1. **Prometheus** – Settings → Data Sources → Add → Prometheus → URL: `http://prometheus:9090`
2. **Loki** – Settings → Data Sources → Add → Loki → URL: `http://loki:3100`

### 7. Import dashboards

- **Node Exporter Full** – Dashboard ID `1860`
- **Loki Logs** – Dashboard ID `13639`

Go to Dashboards → Import → enter the ID → select your Prometheus/Loki datasource.

### 8. Test an alert

```bash
docker run --rm -it wdhif/stress-ng --cpu 0 --cpu-method all --timeout 120s
```

Watch http://localhost:9090/alerts — the `HighSystemLoad` alert should fire within ~2 minutes.

## GitHub Actions CI/CD

### Required secrets (repo Settings → Secrets → Actions)

| Secret | Value |
|--------|-------|
| `SERVER_HOST` | IP or hostname of your server |
| `SERVER_USER` | SSH username (e.g. `ubuntu`) |
| `SSH_PRIVATE_KEY` | Private key matching the server's `authorized_keys` |
| `SERVER_PORT` | SSH port (optional, defaults to 22) |


## Stopping the stack

```bash
docker compose down          # stop containers, keep volumes
docker compose down -v       # stop containers AND delete all data
```
