# Firewall Rules

The Uncomplicated Firewall (UFW) is defined with a default-deny inbound policy.
This means that all incoming packets are blocked, while the outgoing packets are allowed.

## Default Deny Setup

```bash
sudo ufw default deny incoming

sudo ufw default allow outgoing
```

## Explicit Rules Definition

```bash
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
```
