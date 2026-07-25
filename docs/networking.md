
# AGRUPA Cloud — Networking Documentation

## Purpose

This document describes the network layout, exposed services, private access model, and service endpoints used in the AGRUPA Cloud project.

## Known addresses

Public server IP:

```txt id="1fxjjy"
116.203.21.170
```

Private IP used during testing:

```txt id="zgh6k1"
10.0.0.2
```

Storage Box host:

```txt id="5x6agc"
uxxxxxx.your-storagebox.de
```

Storage Box user:

```txt id="g5b2qd"
uxxxxxx
```

## Access model

The intended access model is private-first.

Users access the infrastructure through WireGuard VPN. Internal services should be reached through the private/VPN network whenever possible.

## Service ports

| Service          |      Port | Exposure                          | Purpose                |
| ---------------- | --------: | --------------------------------- | ---------------------- |
| SSH              |        22 | Admin access                      | Server administration  |
| WireGuard        | 51820/UDP | Public                            | VPN entrypoint         |
| Nextcloud        |    80/443 | Private or public depending setup | Web access             |
| Collabora        |      9980 | Private/VPN preferred             | Office editing         |
| MariaDB          |      3306 | Internal only                     | Database               |
| Redis            |      6379 | Internal only                     | Cache and file locking |
| Storage Box SFTP |        22 | External Hetzner service          | Remote storage access  |

## Nextcloud access

Current access during testing:

```txt id="h35htu"
http://10.0.0.2
```

If a domain is later configured, the final URL should use HTTPS.

## Collabora access

Collabora test endpoint:

```txt id="y6bwcv"
http://10.0.0.2:9980/hosting/discovery
```

Expected response:

```txt id="85xv62"
HTTP/1.1 200 OK
```

Nextcloud Office should be configured to use:

```txt id="nzcqm9"
http://10.0.0.2:9980
```

## Storage Box access

SFTP test:

```bash id="093eou"
sftp -P 22 uxxxxxx@uxxxxxx.your-storagebox.de
```

Expected result after login:

```txt id="ng8z3j"
sftp>
```

Inside SFTP:

```bash id="txgh9c"
ls
pwd
bye
```

Expected:

```txt id="oitjdc"
nextcloud-data
Remote working directory: /
```

SSHFS mount:

```bash id="mgcoxh"
sudo sshfs -p 22 uxxxxxx@uxxxxxx.your-storagebox.de:/ /mnt/storagebox \
  -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3
```

## Network validation commands

Check listening ports:

```bash id="ck2tkp"
sudo ss -tulpn
```

Check Docker containers:

```bash id="dlordf"
docker compose ps
```

Check Collabora:

```bash id="oodld6"
curl -I http://10.0.0.2:9980/hosting/discovery
```

Check Nextcloud:

```bash id="xlzlmk"
curl -I http://10.0.0.2
```

Check Storage Box DNS:

```bash id="41zeul"
getent hosts uxxxxxx.your-storagebox.de
```

## Security notes

MariaDB and Redis should remain internal and must not be exposed to the public internet.

Collabora should preferably be reachable through the private/VPN network or behind a reverse proxy with proper HTTPS.

WireGuard is the preferred access method for administration and private service usage.

---

