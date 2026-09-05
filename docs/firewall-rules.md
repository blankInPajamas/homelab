# Firewall Rules

The Uncomplicated Firewall (UFW) is defined with a default-deny inbound policy.
This means that all incoming packets are blocked, while the outgoing packets are allowed. To set this up:

```bash
sudo ufw default deny incoming

sudo ufw default allow outgoing
```


