# Prometheus Monitoring Stack

A Docker Compose setup for monitoring Proxmox VE (PVE) hosts and Plex Media Server using Prometheus and custom exporters.

## Services

### Prometheus
- **Image**: `prom/prometheus`
- **Port**: `9090`
- **Purpose**: Main monitoring and time-series database
- **Web UI**: http://localhost:9090

### Prometheus PVE Exporter
- **Image**: `prompve/prometheus-pve-exporter`
- **Purpose**: Exports metrics from Proxmox VE clusters
- **Port**: `9221` (internal)

### Prometheus Plex Exporter
- **Image**: `ghcr.io/jsclayton/prometheus-plex-exporter`
- **Purpose**: Exports metrics from Plex Media Server
- **Port**: `9000` (internal)

## Prerequisites

- Docker and Docker Compose installed
- Access to Proxmox VE clusters (if monitoring PVE)
- Access to Plex Media Server (if monitoring Plex)

## Configuration

### 1. Plex Configuration

Create a `plex.env` file with your Plex server details:
```env
PLEX_SERVER=http://your-plex-server:32400
PLEX_TOKEN=your-plex-token
```

**Getting your Plex token:**
- Follow the [official Plex documentation](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/)

### 2. Proxmox VE Configuration

The `pve.yml` file contains credentials for each Proxmox cluster. Each cluster is configured as a separate module:

```yaml
alki:
  user: prometheus@pve
  token_name: "exporter"
  token_value: "your-token-value"
yvr:
  user: prometheus@pve
  token_name: "exporter"
  token_value: "your-token-value"
```

To add a new Proxmox cluster, add a new module section and update `prometheus.yml` with a corresponding job.

**Creating a Proxmox API token:**
1. Log in to your Proxmox web interface
2. Navigate to Datacenter → Permissions → API Tokens
3. Create a new token for the `prometheus@pve` user
4. Copy the token ID and secret value

### 3. Prometheus Scrape Configuration

The `prometheus.yml` file configures what Prometheus monitors:

- **prometheus**: Self-monitoring (scrape interval: 5s)
- **pve-alki**: Proxmox cluster at `alki.kekra.net`
- **pve-yvr**: Proxmox cluster at `yvr.kekra.net`
- **plex**: Plex Media Server metrics

Edit `prometheus.yml` to match your infrastructure:
- Update PVE target cluster names
- Modify scrape intervals if needed
- Add or remove job configurations

## Usage

### Start the Stack

```bash
docker compose up -d
```

### Stop the Stack

```bash
docker compose down
```

### View Logs

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f prometheus
docker compose logs -f prometheus-pve-exporter
docker compose logs -f prometheus-plex-exporter
```

### Restart Services

```bash
docker compose restart
```

## Accessing Prometheus

Once running, access the Prometheus web UI at:
- http://localhost:9090

From here you can:
- Query metrics using PromQL
- View targets status
- Create graphs and dashboards
- Check service health

## Data Persistence

Prometheus data is stored in a Docker volume named `prometheus-data`, ensuring metrics persist across container restarts.

## Monitored Metrics

### Proxmox VE Metrics
- CPU usage
- Memory usage
- Disk I/O
- Network traffic
- VM/Container status and resource usage

### Plex Metrics
- Active streams
- Library statistics
- Bandwidth usage
- Transcode sessions

## Troubleshooting

### Check Service Status
```bash
docker compose ps
```

### Verify Prometheus Targets
1. Open http://localhost:9090/targets
2. Check that all targets show as "UP"
3. If targets are down, check the exporter logs

### Common Issues

**PVE Exporter not connecting:**
- Verify `pve.yml` credentials are correct
- Ensure Proxmox API token has proper permissions
- Check network connectivity to Proxmox clusters

**Plex Exporter not connecting:**
- Verify `plex.env` server URL and token
- Ensure Plex server is accessible from Docker network
- Check Plex token validity

## Files Structure

```
.
├── compose.yaml              # Docker Compose service definitions
├── prometheus.yml            # Prometheus scrape configuration
├── pve.yml                   # Proxmox VE credentials (gitignored)
├── plex.env                 # Plex server configuration (gitignored)
└── README.md                # This file
```

**Current PVE Modules:**
- `alki` - Proxmox cluster at alki.kekra.net
- `yvr` - Proxmox cluster at yvr.kekra.net

## Security Notes

- Never commit `plex.env` or `pve.yml` to version control (they contain sensitive credentials)
- Use API tokens with minimal required permissions
- Consider running Prometheus behind a reverse proxy with authentication for production use
- Restrict network access to the Prometheus port (9090) if exposed

## Further Configuration

- For alerting, consider adding Alertmanager
- For visualization, integrate with Grafana
- For long-term storage, configure remote write to a time-series database
