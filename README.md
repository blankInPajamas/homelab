# Homelab Infrastructure

Self-hosted, multi-service infrastructure orchestrated with Docker Compose, 
fronted by Nginx as a reverse proxy, with manually designed firewall rules.

## Services currently running
- Pi-hole (DNS / ad-blocking)
- Audiobookshelf (audiobook server)

## Architecture
See `docs/network-architecture.png` (generated via `docs/generate_network_diagram.py`)

## Quickstart

```bash   
cd homelab/
docker compose up -d
```

## Project Structure

```text
homelab/
├── docker-compose.yml
├── .env.example
├── .gitignore
├── README.md
├── nginx/
│   └── sites-available/
│       ├── pihole.conf
│       └── audiobookshelf.conf
├── docs/
│   ├── network-architecture.png
│   ├── generate_network_diagram.py
│   ├── firewall-rules.md          (placeholder for now — Phase 3 not done yet)
│   └── disaster-recovery.md       (placeholder for now — Phase 7 not done yet)
└── scripts/                        (empty for now, for future automation)
```

## Roadmap
- [x] Certbot / real SSL
- [ ] WireGuard VPN
- [ ] Firewall rules (iptables/ufw)
- [ ] Observability stack (Prometheus, Grafana, Loki)
- [ ] Dashboard (Homepage/Homarr, Portainer)
- [ ] Backups (Restic/Duplicati)

