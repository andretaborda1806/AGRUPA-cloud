
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

# docs/disaster-recovery.md

# AGRUPA Cloud — Disaster Recovery Guide

## Purpose

This document describes recovery procedures for major failure scenarios.

The system has already gone through a real recovery involving:

* missing/incomplete Nextcloud configuration;
* empty active MariaDB database;
* Storage Box not mounted;
* invalid Nextcloud data directory;
* restored MariaDB backup;
* restored Nextcloud `config.php`;
* remounted Hetzner Storage Box.

## Scenario 1 — Server rebooted and Nextcloud does not start

### Symptoms

```txt id="8eizgv"
Your data directory is invalid
```

or Nextcloud asks to create an admin account.

### Checks

```bash id="d23o5j"
cd /opt/agrupa-cloud
findmnt /mnt/storagebox
df -h /mnt/storagebox
docker exec -u www-data -it nextcloud php occ status
```

If `/mnt/storagebox` shows `/dev/sda1`, the Storage Box is not mounted.

### Recovery

```bash id="6szx1l"
docker compose stop nextcloud

sudo sshfs -p 22 uxxxxxx@uxxxxxx.your-storagebox.de:/ /mnt/storagebox \
  -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3

docker compose up -d nextcloud
docker exec -u www-data -it nextcloud php occ status
```

Expected:

```txt id="x5iofi"
installed: true
```

## Scenario 2 — Storage Box is not mounted

### Check

```bash id="ulgca5"
findmnt /mnt/storagebox
df -h /mnt/storagebox
```

Incorrect:

```txt id="r4npzq"
/dev/sda1 ... /
```

Correct:

```txt id="va895a"
uxxxxxx@uxxxxxx.your-storagebox.de:/ ... /mnt/storagebox
```

### Recovery

```bash id="6ou8ye"
docker compose stop nextcloud

sudo mv /mnt/storagebox /mnt/storagebox.local-wrong-$(date +%Y%m%d-%H%M%S)
sudo mkdir -p /mnt/storagebox

sudo sshfs -p 22 uxxxxxx@uxxxxxx.your-storagebox.de:/ /mnt/storagebox \
  -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3
```

Validate:

```bash id="du08ri"
sudo ls -la /mnt/storagebox/nextcloud-data | head -40
sudo find /mnt/storagebox/nextcloud-data -maxdepth 1 -name ".ncdata" -type f
```

## Scenario 3 — Nextcloud configuration is lost

### Symptoms

Nextcloud asks to create a new admin user.

### Check

```bash id="2lvihp"
sudo ls -la /var/lib/docker/volumes/agrupa-cloud_nextcloud_config/_data
sudo grep -E "'installed'|'dbhost'|'dbname'|'dbuser'|'datadirectory'" \
  /var/lib/docker/volumes/agrupa-cloud_nextcloud_config/_data/config.php
```

If `installed => true` is missing, restore the config.

### Restore config

```bash id="ixmx47"
docker compose down

sudo rm -rf /var/lib/docker/volumes/agrupa-cloud_nextcloud_config/_data/*

sudo tar -xzf /opt/agrupa-cloud/backups/backup-20260713-113323/nextcloud-config.tar.gz \
  -C /var/lib/docker/volumes/agrupa-cloud_nextcloud_config/_data \
  --strip-components=1
```

Start:

```bash id="mc1n26"
docker compose up -d
docker exec -u www-data -it nextcloud php occ status
```

## Scenario 4 — MariaDB is empty

### Symptoms

Error:

```txt id="iqw2qj"
Table 'nextcloud.oc_users' doesn't exist
```

### Check

```bash id="o4133r"
sudo ls -la /mnt/HC_Volume_106259255/mariadb/nextcloud | head -80
```

If only `db.opt` exists, the database is empty.

### Restore MariaDB

```bash id="v8p4i1"
cd /opt/agrupa-cloud
docker compose down

sudo rm -rf /mnt/HC_Volume_106259255/mariadb/*
sudo cp -a /mnt/backup/mariadb/. /mnt/HC_Volume_106259255/mariadb/

docker compose up -d mariadb redis
sleep 20
docker compose up -d nextcloud
docker exec -u www-data -it nextcloud php occ status
```

## Scenario 5 — Office documents download instead of opening

### Check

```bash id="aum052"
docker exec -u www-data -it nextcloud php occ app:list | grep richdocuments
curl -I http://10.0.0.2:9980/hosting/discovery
```

### Recovery

```bash id="waumqv"
docker exec -u www-data -it nextcloud php occ app:install richdocuments
docker exec -u www-data -it nextcloud php occ app:enable richdocuments
```

Then configure in the web UI:

```txt id="wjlcav"
Admin settings → Office → Use your own server
```

URL:

```txt id="pu2efd"
http://10.0.0.2:9980
```

## Scenario 6 — Full rebuild from backups

High-level recovery order:

```txt id="10ib1s"
1. Recreate server
2. Install Docker and Docker Compose
3. Restore /opt/agrupa-cloud
4. Mount Storage Box
5. Restore MariaDB
6. Restore Nextcloud config
7. Start MariaDB and Redis
8. Start Nextcloud
9. Validate occ status
10. Start Collabora and WireGuard
```

Validation commands:

```bash id="n6wkcd"
docker compose ps
docker exec -u www-data -it nextcloud php occ status
curl -I http://10.0.0.2:9980/hosting/discovery
df -h /mnt/storagebox
```
