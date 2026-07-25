

# AGRUPA Cloud — Operational Runbook

## Purpose

This runbook provides the standard operational procedures for starting, stopping, validating, and recovering the AGRUPA Cloud services.

It should be used whenever the server is rebooted, containers are restarted, or Nextcloud is not responding correctly.

## Project location

```bash id="lfoau8"
/opt/agrupa-cloud
```

Always start from:

```bash id="ofkdkk"
cd /opt/agrupa-cloud
```

## Standard health check

Run:

```bash id="lby4xg"
docker compose ps
```

Then:

```bash id="b4ctsh"
docker exec -u www-data -it nextcloud php occ status
```

Expected Nextcloud status:

```txt id="r0xix0"
installed: true
maintenance: false
needsDbUpgrade: false
```

Check Storage Box:

```bash id="ekplvz"
findmnt /mnt/storagebox
df -h /mnt/storagebox
```

Expected Storage Box output:

```txt id="8yg1lw"
u628575@u628575.your-storagebox.de:/ ... /mnt/storagebox
```

## Start procedure

### 1. Check Storage Box

```bash id="8a5lyt"
findmnt /mnt/storagebox
df -h /mnt/storagebox
sudo find /mnt/storagebox/nextcloud-data -maxdepth 1 -name ".ncdata" -type f
```

If the mount is correct, continue.

### 2. Start database and cache first

```bash id="stj6vc"
docker compose up -d mariadb redis
```

Wait a few seconds:

```bash id="g2f3ex"
sleep 20
```

### 3. Start Nextcloud

```bash id="jhfb9g"
docker compose up -d nextcloud
```

Validate:

```bash id="1fli35"
docker exec -u www-data -it nextcloud php occ status
```

### 4. Start the remaining services

```bash id="56v2vi"
docker compose up -d
```

Check:

```bash id="9uzprh"
docker compose ps
```

## Stop procedure

Stop all services:

```bash id="e2r8gt"
docker compose down
```

Stop only Nextcloud:

```bash id="nkbjq7"
docker compose stop nextcloud
```

Stop only Collabora:

```bash id="5ua6ic"
docker compose stop collabora
```

## Restart procedure

Recommended safe restart:

```bash id="axgycm"
cd /opt/agrupa-cloud
docker compose stop nextcloud collabora
findmnt /mnt/storagebox
df -h /mnt/storagebox
docker compose up -d mariadb redis
docker compose up -d nextcloud
docker compose up -d collabora
docker compose ps
```

Validate:

```bash id="dzvtke"
docker exec -u www-data -it nextcloud php occ status
curl -I http://10.0.0.2:9980/hosting/discovery
```

## Logs

Nextcloud:

```bash id="z83ujq"
docker compose logs --tail=100 nextcloud
```

MariaDB:

```bash id="4vdo19"
docker compose logs --tail=100 mariadb
```

Redis:

```bash id="0rndmp"
docker compose logs --tail=100 redis
```

Collabora:

```bash id="s7w7og"
docker compose logs --tail=100 collabora
```

WireGuard:

```bash id="j43k7o"
docker compose logs --tail=100 wireguard
```

## Storage Box manual mount

If the Storage Box is not mounted:

```bash id="z1eukd"
docker compose stop nextcloud
sudo sshfs -p 22 u628575@u628575.your-storagebox.de:/ /mnt/storagebox \
  -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3
```

Validate:

```bash id="98kzww"
df -h /mnt/storagebox
sudo ls -la /mnt/storagebox/nextcloud-data | head -40
```

## Validate Collabora

```bash id="tws0d8"
curl -I http://10.0.0.2:9980/hosting/discovery
```

Expected:

```txt id="jijmqs"
HTTP/1.1 200 OK
```

## Validate Nextcloud Office

Check app:

```bash id="p8z1lq"
docker exec -u www-data -it nextcloud php occ app:list | grep richdocuments
```

Enable if needed:

```bash id="81oebu"
docker exec -u www-data -it nextcloud php occ app:install richdocuments
docker exec -u www-data -it nextcloud php occ app:enable richdocuments
```

## Common emergency command set

```bash id="f4cfuv"
cd /opt/agrupa-cloud
findmnt /mnt/storagebox
df -h /mnt/storagebox
docker compose ps
docker exec -u www-data -it nextcloud php occ status
docker compose logs --tail=80 nextcloud
```
