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

## Enable

```bash
sudo ufw enable
```

This will enable the UFW.

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
