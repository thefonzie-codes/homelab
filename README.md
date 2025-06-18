# Homelab Setup Documentation

A self-hosted homelab setup for Obsidian sync, photo storage, and learning Kubernetes/Docker.

## Prerequisites

### Software Requirements
- Ubuntu Server (target) or Linux Mint (development)
- Docker and Docker Compose
- Tailscale for secure remote access
- Git for configuration management

### Initial Setup Commands

```bash
# Create project structure
mkdir ~/homelab
cd ~/homelab
git init
mkdir -p {docker-compose,kubernetes,docs,scripts,monitoring}
```

### Docker Installation

```bash
# Update system
sudo apt update

# Install Docker
sudo apt install docker.io docker-compose-plugin -y

# Add user to docker group
sudo usermod -aG docker $USER

# Log out and back in, then test
docker --version
docker ps
```

### Tailscale Installation

```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Start Tailscale (will provide auth URL)
sudo tailscale up

# Get your IP and hostname
tailscale ip -4
sudo tailscale whois $(tailscale ip -4) | grep Name
```

## Services

### 1. Obsidian Sync Server (CouchDB)

**Location:** `~/homelab/docker-compose/obsidian-sync.yml`

**Configuration Files:**
- `docker-compose/obsidian-sync.yml` - Docker Compose configuration
- `docker-compose/config/local.ini` - CouchDB settings

**Default Credentials:**
- Username: `admin`
- Password: `obsidian_admin_password_change_me`
- Database: `obsidian-vault`

**Deployment:**

```bash
cd ~/homelab/docker-compose

# Start CouchDB
docker compose -f obsidian-sync.yml up -d

# Check status
docker ps

# Create Obsidian database
curl -u admin:obsidian_admin_password_change_me -X PUT http://localhost:5984/obsidian-vault

# Test web interface
# Visit: http://localhost:5984/_utils
```

**Mobile Access via Tailscale:**

```bash
# Get your Tailscale hostname
sudo tailscale whois $(tailscale ip -4) | grep Name
# Example result: kagenousama-legion-pro-5-16irx8.tail2bbd46.ts.net

# Mobile URL: http://YOUR_TAILSCALE_HOSTNAME:5984
```

**Obsidian Mobile Setup:**
1. Install "Self-hosted LiveSync" community plugin
2. Configure with:
   - URI: `http://your-tailscale-hostname.ts.net:5984`
   - Database: `obsidian-vault`
   - Username: `admin`
   - Password: `obsidian_admin_password_change_me`

### 2. Photo Storage (Nextcloud) - TODO

Future service for Google Photos replacement.

### 3. Monitoring Stack - TODO

Planned: Prometheus + Grafana for system monitoring.

### 4. Reverse Proxy - TODO

Planned: Traefik for SSL termination and routing.

## Network Configuration

### Local Access
- CouchDB Web Interface: `http://localhost:5984/_utils`
- All services accessible on local network

### Remote Access (Tailscale)
- Secure VPN access to all services
- HTTPS certificates available on paid Tailscale plans
- Mobile access from anywhere

### Ports Used
- `5984` - CouchDB (Obsidian sync)
- `8080` - Nextcloud (planned)
- `3000` - Grafana (planned)
- `9090` - Prometheus (planned)

## Troubleshooting

### Docker Issues

```bash
# Check Docker daemon
sudo systemctl status docker

# Fix permissions
sudo usermod -aG docker $USER
newgrp docker

# Use newer compose syntax
docker compose instead of docker-compose
```

### Tailscale Issues

```bash
# Check Tailscale status
sudo tailscale status

# Test connectivity
curl -u admin:password http://tailscale-hostname:5984/

# Restart Tailscale
sudo tailscale down
sudo tailscale up
```

### Obsidian Sync Issues

1. Verify CouchDB is running: `docker ps`
2. Test database exists: `curl -u admin:password http://localhost:5984/_all_dbs`
3. Check Tailscale connectivity from mobile
4. Verify LiveSync plugin settings
5. Look for plugin error logs in Obsidian mobile

## Migration from Development to Production

### Development (Linux Mint Desktop)
1. Test all configurations locally
2. Document all setup steps
3. Create infrastructure-as-code configs
4. Test backup/restore procedures

### Production (ThinkCentre Ubuntu Server)
1. Fresh Ubuntu Server installation
2. Install Docker and Tailscale
3. Clone configuration repository
4. Deploy using documented procedures
5. Copy persistent data volumes

### Data Migration

```bash
# Export data from development
docker compose -f obsidian-sync.yml down
tar -czf homelab-backup.tar.gz data/

# Import to production
scp homelab-backup.tar.gz user@production-server:~/
ssh user@production-server
tar -xzf homelab-backup.tar.gz
docker compose -f obsidian-sync.yml up -d
```

## Security Considerations

### Default Passwords
**IMPORTANT:** Change all default passwords before production use!

- CouchDB admin password
- Any database passwords
- SSH key-based authentication

### Network Security
- Tailscale provides encrypted mesh networking
- No ports exposed to public internet
- Consider fail2ban for SSH protection

### Backup Strategy
- Regular exports of container volumes
- Configuration files in Git
- Document restore procedures

## Future Enhancements

### Planned Services
- [ ] Nextcloud for photo storage
- [ ] Prometheus + Grafana monitoring
- [ ] Traefik reverse proxy with SSL
- [ ] Automated backups
- [ ] CI/CD pipeline for deployments

### Infrastructure Improvements
- [ ] Kubernetes migration (k3s)
- [ ] Infrastructure as Code (Ansible/Terraform)
- [ ] Automated SSL certificate management
- [ ] High availability configurations

## Resources

- [Tailscale Documentation](https://tailscale.com/kb/)
- [CouchDB Documentation](https://docs.couchdb.org/)
- [Obsidian LiveSync Plugin](https://github.com/vrtmrz/obsidian-livesync)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## Support

For issues or questions:
1. Check the troubleshooting section
2. Review Docker and Tailscale logs
3. Verify network connectivity
4. Document any new solutions found