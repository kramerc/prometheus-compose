# Prometheus Monitoring Stack

A Docker Compose setup for monitoring Proxmox VE (PVE) hosts, Jellyfin Media Server, and Plex Media Server using Prometheus and custom exporters.

## Services

### Prometheus
- **Image**: `prom/prometheus`
- **Port**: `9090`
- **Purpose**: Main monitoring and time-series database
- **Web UI**: http://localhost:9090

### Prometheus Jellyfin Exporter
- **Image**: `rebelcore/jellyfin-exporter:latest`
- **Purpose**: Exports metrics from Jellyfin Media Server including user playback status
- **Port**: `9594` (internal)

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
- Access to Jellyfin Media Server (if monitoring Jellyfin)
- Access to Plex Media Server (if monitoring Plex)

## Configuration

### 1. Jellyfin Configuration

Create a `jellyfin.env` file with your Jellyfin server details:
```env
JELLYFIN_ADDRESS=http://your-jellyfin-server:8096
JELLYFIN_TOKEN=your-api-key
```

**Getting your Jellyfin API key:**
1. Log in to your Jellyfin web interface
2. Navigate to Dashboard → Advanced → API Keys
3. Click the "+" button to create a new API key
4. Give it a descriptive name (e.g., "Prometheus Exporter")
5. Copy the generated API key

### 2. Plex Configuration

Create a `plex.env` file with your Plex server details:
```env
PLEX_SERVER=http://your-plex-server:32400
PLEX_TOKEN=your-plex-token
```

**Getting your Plex token:**
- Follow the [official Plex documentation](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/)

### 3. Proxmox VE Configuration

Create a `pve.yml` file with credentials for each Proxmox cluster. Each cluster is configured as a separate module.

**Option 1: Token-based authentication (recommended):**
```yaml
alki:
  user: prometheus@pve
  token_name: "exporter"
  token_value: "your-token-value"
  verify_ssl: false
```

**Option 2: Password-based authentication (alternative):**
```yaml
cluster_name:
  user: prometheus@pve
  password: "your-password"
  verify_ssl: false
```

To add a new Proxmox cluster:
1. Add a new module section to `pve.yml`
2. Create the corresponding user and API token in Proxmox
3. Update `prometheus.yml` with a corresponding scrape job

**Creating a Proxmox API token:**
1. Log in to your Proxmox web interface
2. Navigate to Datacenter → Permissions → Users
3. Create or verify the `prometheus@pve` user exists
4. Assign the `PVESysAdmin` role to the user at the Datacenter level
5. Navigate to Datacenter → Permissions → API Tokens
6. Create a new token for the `prometheus@pve` user with the token ID `exporter`
7. Ensure the token also has the `PVESysAdmin` role assigned
8. Copy the generated token secret value

### 4. Prometheus Scrape Configuration

The `prometheus.yml` file configures what Prometheus monitors:

- **prometheus**: Self-monitoring (scrape interval: 5s)
- **jellyfin**: Jellyfin Media Server metrics (scrape interval: 15s)
- **pve-alki**: Proxmox cluster at `alki.kekra.net` (scrape interval: 15s)
- **plex**: Plex Media Server metrics (scrape interval: 15s)

Edit `prometheus.yml` to match your infrastructure:
- Update target hostnames and cluster names
- Modify scrape intervals if needed
- Add or remove job configurations
- For PVE jobs, configure the `module`, `cluster`, and `node` parameters to match your `pve.yml` setup

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
docker compose logs -f prometheus-jellyfin-exporter
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

### Jellyfin Metrics
- User playback status (playing/paused)
- Currently playing items (name, type, series/episode info)
- Active sessions and clients
- Device information

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
├── jellyfin.env              # Jellyfin server configuration (gitignored)
├── plex.env                  # Plex server configuration (gitignored)
├── prometheus.yml            # Prometheus scrape configuration
├── pve.yml                   # Proxmox VE credentials (gitignored)
└── README.md                 # This file
```

**Current PVE Modules:**
- `alki` - Proxmox cluster at alki.kekra.net

## Security Notes

- Never commit `jellyfin.env`, `plex.env` or `pve.yml` to version control (they contain sensitive credentials)
- Use API tokens with minimal required permissions
- Consider running Prometheus behind a reverse proxy with authentication for production use
- Restrict network access to the Prometheus port (9090) if exposed

## Further Configuration

- For alerting, consider adding Alertmanager
- For visualization, integrate with Grafana
- For long-term storage, configure remote write to a time-series database
