# Custom Homepage Fork with Snowflake Widget

This is a custom fork of [Homepage](https://github.com/gethomepage/homepage) that includes a Tor Snowflake Proxy widget.

## What's Different

- **Snowflake Widget**: Native widget for monitoring Tor Snowflake Proxy metrics
- **Auto-sync**: Daily automatic sync with upstream Homepage to get latest features
- **Pre-built Images**: Automatically built Docker images published to GitHub Container Registry

## Using This Fork

### Option 1: Use Pre-built Docker Image (Recommended)

Update your `docker-compose.yml`:

```yaml
homepage:
  image: ghcr.io/qudiqudi/homepage:snowflake
  container_name: homepage
  # ... rest of your config
```

The image is automatically rebuilt when:
- Upstream Homepage releases updates (daily sync)
- Changes are pushed to this fork

### Option 2: Build Yourself

Clone and build:

```bash
git clone https://github.com/qudiqudi/homepage.git
cd homepage
git checkout feat/snowflake-widget
docker build -t homepage-custom .
```

## Snowflake Widget Configuration

Add to your `services.yaml`:

```yaml
- Tools:
  - Snowflake Proxy:
      icon: mdi-snowflake
      description: Tor Snowflake Proxy
      href: https://snowflake.torproject.org/
      widget:
        type: snowflake
        url: http://172.17.0.1:9199  # Or host.docker.internal:9199
```

### Docker Compose Setup

Full example with Snowflake proxy:

```yaml
# Snowflake Proxy
snowflake-proxy:
  image: containers.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake:latest
  container_name: snowflake-proxy
  network_mode: host
  security_opt:
    - no-new-privileges:true
  restart: unless-stopped
  command: ["-metrics", "-metrics-address", "0.0.0.0", "-metrics-port", "9199"]

# Homepage with Snowflake Widget
homepage:
  image: ghcr.io/qudiqudi/homepage:snowflake
  container_name: homepage
  networks:
    - bridge_network
  extra_hosts:
    - "host.docker.internal:host-gateway"  # For Linux
  volumes:
    - ./homepage/config:/app/config
  restart: unless-stopped
```

## How Auto-Sync Works

### Daily Sync Workflow

- Runs daily at 2 AM UTC
- Fetches latest changes from `gethomepage/homepage:dev`
- Merges into `feat/snowflake-widget` branch
- Triggers Docker image rebuild
- If merge conflicts occur, creates a GitHub issue

### Manual Sync

Trigger manually via GitHub Actions:
1. Go to https://github.com/qudiqudi/homepage/actions
2. Select "Sync with Upstream Homepage"
3. Click "Run workflow"

### Handling Conflicts

If auto-sync detects conflicts:

```bash
git clone https://github.com/qudiqudi/homepage.git
cd homepage
git checkout feat/snowflake-widget
git fetch upstream
git merge upstream/dev
# Resolve conflicts in your editor
git add .
git commit
git push origin feat/snowflake-widget
```

## Widget Features

The Snowflake widget displays:
- **Connections**: Total successful connections served
- **Inbound**: Download traffic from clients
- **Outbound**: Upload traffic to Tor network
- **Countries**: Number of unique countries helped

## Staying Updated

Your Homepage will automatically stay up-to-date with:
- ✅ Latest Homepage features and bug fixes
- ✅ Snowflake widget improvements
- ✅ Security patches

No manual intervention needed unless merge conflicts occur.

## Image Tags

- `snowflake` - Latest build from feat/snowflake-widget (recommended)
- `feat-snowflake-widget` - Same as above
- `feat-snowflake-widget-<sha>` - Specific commit builds

## Support

- **Upstream Homepage**: https://github.com/gethomepage/homepage
- **This Fork**: https://github.com/qudiqudi/homepage
- **Snowflake**: https://snowflake.torproject.org/

## Why This Fork Exists

The Snowflake widget PR was not accepted by the upstream project. This fork maintains the widget while staying synchronized with upstream Homepage development.
