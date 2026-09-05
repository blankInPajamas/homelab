# Homelab Infrastructure

Self-hosted, multi-service infrastructure orchestrated with Docker Compose, 
fronted by Nginx as a reverse proxy, with manually designed firewall rules.

## Services currently running
- Pi-hole (DNS / ad-blocking)
- Audiobookshelf (audiobook server)

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

## Setting Up
Everything in this repo is version-controlled, but several host-level pieces are **not** tracked by Git and must be recreated manually on every fresh VM/instance:

- Nginx symlinks (`sites-available` → repo, `sites-enabled` → `sites-available`)
- Self-signed SSL certificates
- `/etc/hosts` entries
- `.env` file
- The `systemd-resolved` port 53 fix (required before Pi-hole can bind DNS)

Run through this checklist top to bottom on any new instance before expecting `docker compose up -d` or Nginx to work correctly.

### 1. Clone the repo
```bash
git clone <your-repo-url> ~/homelab
cd ~/homelab
```
### 2. Fix the systemd-resolved port 53 conflict
Pi-hole needs to bind host port 53 for DNS, which `systemd-resolved`'s stub listener occupies by default on Ubuntu.

```bash
sudo nano /etc/systemd/resolved.conf
```

In the `[Resolve]` section, set:

```
DNSStubListener=no
```

Then:

```bash
sudo rm /etc/resolv.conf
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
sudo systemctl restart systemd-resolved
```

Confirm port 53 is free:

```bash
sudo ss -tulpn | grep :53
```

(should return nothing)


### 3. Recreate `.env`
Not tracked in Git (see `.env.example` for the template)
```bash
cat > ~/homelab/.env << 'EOF'
TZ=Asia/Dhaka
EOF
```


### 4. Bring up the containers

```bash
cd ~/homelab
docker compose up -d
docker ps
```
Confirm both `pihole` and `audiobookshelf` show `Up`.


### 5. Symlink Nginx configs from the repo

The actual config files live in this repo at `nginx/sites-available/`.
Nginx itself reads from `/etc/nginx/sites-available/` and `/etc/nginx/sites-enabled/`, so both need to be symlinked back to the repo copies — never copy, always symlink, so editing the file in the repo is editing the live config.

```bash
sudo ln -s ~/homelab/nginx/sites-available/pihole.conf /etc/nginx/sites-available/pihole.conf
sudo ln -s ~/homelab/nginx/sites-available/audiobookshelf.conf /etc/nginx/sites-available/audiobookshelf.conf

sudo ln -s /etc/nginx/sites-available/pihole.conf /etc/nginx/sites-enabled/pihole.conf
sudo ln -s /etc/nginx/sites-available/audiobookshelf.conf /etc/nginx/sites-enabled/audiobookshelf.conf
```

Verify:

```bash
ls -la /etc/nginx/sites-available/
ls -la /etc/nginx/sites-enabled/
```
Both should show arrows pointing back into `~/homelab/nginx/sites-available/`.


### 6. Generate self-signed SSL certificates

Certs are intentionally **not** committed to Git (private key material).
Regenerate them on every fresh instance:

```bash
sudo mkdir -p /etc/nginx/ssl

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/pihole-selfsigned.key \
  -out /etc/nginx/ssl/pihole-selfsigned.crt
# IMPORTANT: Common Name (CN) must be exactly: pihole.local

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/audiobookshelf-selfsigned.key \
  -out /etc/nginx/ssl/audiobookshelf-selfsigned.crt
# IMPORTANT: Common Name (CN) must be exactly: audiobookshelf.local
```

Verify the CN is correct before moving on (a mismatched CN causes
`certificate subject name does not match target hostname` errors):

```bash
openssl x509 -in /etc/nginx/ssl/pihole-selfsigned.crt -noout -subject
openssl x509 -in /etc/nginx/ssl/audiobookshelf-selfsigned.crt -noout -subject
```


### 7. Add `/etc/hosts` entries on the VM

```bash
sudo tee -a /etc/hosts << 'EOF'
127.0.1.1 pihole.local
127.0.1.1 audiobookshelf.local
EOF
```


### 8. Test and reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx

curl -Ik https://pihole.local
curl -Ik https://audiobookshelf.local
```

Both should return `200 OK` (Pi-hole may return a `301` to `/admin/`, which is correct).


### 9. Access from the Windows host browser

Get the VM's IP (bridged network mode required — NAT mode is not directly reachable from the host without port forwarding):

```bash
ip a
```

Look for the address under your bridged adapter (e.g. `enp0s3`), typically in your home LAN's range (`192.168.x.x`).


On **Linux**, edit /etc/hosts file using `sudo nano /etc/hosts` and add.
On **Windows**, edit `C:\Windows\System32\drivers\etc\hosts` (Notepad run as Administrator, "All Files" filter to see the extensionless file):

```
<vm-ip>  pihole.local
<vm-ip>  audiobookshelf.local
```

Verify:

```bash
ping pihole.local
curl.exe -Ik https://pihole.local
```

In PowerShell (Windows):

```powershell
ping pihole.local
curl.exe -Ik https://pihole.local
```

Then open `https://pihole.local` and `https://audiobookshelf.local` directly in a browser. Expect a self-signed certificate warning — this is expected and correct behavior for a self-signed cert; click through ("Advanced" → "Proceed").

