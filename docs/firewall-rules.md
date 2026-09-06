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
## Check

To check whether the default-deny is working and the explicit rules have been set or not:

```bash
sudo ufw status verbose
```
The output should look like this:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80                         ALLOW IN    Anywhere
443                        ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
80 (v6)                    ALLOW IN    Anywhere (v6)
443 (v6)                   ALLOW IN    Anywhere (v6)
```

## Logging on UFW

```bash
sudo ufw logging medium
```

Logs are written to `/var/log/ufw.log`.

## Block Internal Access Ports

```bash
sudo ufw deny 8081
sudo ufw deny 13378
```

## Rate limiting

```bash
sudo ufw limit ssh
```

## Enable
```bash
sudo ufw enable
```

This will enable the UFW.


## Verification

We can confirm via direct testing from a separate machine on the network:

- `https://pihole.local` and `https://audiobookshelf.local` should be reachable
- `http://<host-ip>:8081` and `http://<host-ip>:13378` should be blocked after DOCKER-USER chain fix applied
