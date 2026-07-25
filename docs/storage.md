
# AGRUPA Cloud — Storage Documentation

## Purpose

This document explains how storage is handled in the AGRUPA Cloud project, with special focus on the Hetzner Storage Box integration.

The Storage Box is a critical part of the system because it stores the Nextcloud user data directory.

## Storage design

Nextcloud separates application code, configuration, database data, and user files.

In this project, the important storage paths are:

```txt id="6w385i"
Project directory:        /opt/agrupa-cloud
MariaDB data:             /mnt/HC_Volume_106259255/mariadb
Nextcloud config volume:  /var/lib/docker/volumes/agrupa-cloud_nextcloud_config/_data
Nextcloud user data:      /mnt/storagebox/nextcloud-data
Storage Box mountpoint:   /mnt/storagebox
```

Inside the Nextcloud container, the user data directory is:

```txt id="4v2bp6"
/var/www/html/data
```

This is mapped from the host path:

```txt id="osx60u"
/mnt/storagebox/nextcloud-data
```

## Docker volume mapping

The Nextcloud service uses this bind mount:

```yaml id="fcxg9s"
- /mnt/storagebox/nextcloud-data:/var/www/html/data
```

This means the container expects `/mnt/storagebox/nextcloud-data` to already exist and contain a valid Nextcloud data directory.

## Storage Box access details

The Hetzner Storage Box is accessed through SFTP/SSHFS.

Storage Box user:

```txt id="fi33sp"
u628575
```

Storage Box host:

```txt id="d5f0qj"
u628575.your-storagebox.de
```

Remote path used:

```txt id="m1ycyx"
/
```

Local mountpoint:

```txt id="t7ku6r"
/mnt/storagebox
```

Manual mount command:

```bash id="ak4twu"
sudo sshfs -p 22 u628575@u628575.your-storagebox.de:/ /mnt/storagebox \
  -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3
```

## Validating the mount

Before starting Nextcloud, always verify that the Storage Box is mounted correctly.

```bash id="d7zuxl"
findmnt /mnt/storagebox
df -h /mnt/storagebox
```

Correct output should show something like:

```txt id="93b1ci"
u628575@u628575.your-storagebox.de:/  1.0T  204M  1.0T   1% /mnt/storagebox
```

Incorrect output looks like this:

```txt id="exglfb"
/dev/sda1 ... /
```

If `/dev/sda1` appears, `/mnt/storagebox` is only a local folder on the VPS, not the real Storage Box.

## Expected Nextcloud data directory contents

A valid Nextcloud data directory should contain:

```txt id="bptkx1"
.ncdata
.htaccess
index.html
nextcloud.log
appdata_...
AGRUPA-admin
Dina
Izequiel
Jose Filipe
__groupfolders
```

The `.ncdata` file is required. If it is missing, Nextcloud may fail with:

```txt id="pl860c"
Your data directory is invalid.
Ensure there is a file called ".ncdata" in the root of the data directory.
```

## Important warning

Do not manually create `.ncdata` before confirming the Storage Box is mounted.

Creating `.ncdata` in a local empty folder may hide the real problem and cause Nextcloud to start against an empty data directory.

## Safe startup order

Before starting Nextcloud:

```bash id="3k7k7j"
findmnt /mnt/storagebox
df -h /mnt/storagebox
sudo find /mnt/storagebox/nextcloud-data -maxdepth 1 -name ".ncdata" -type f
```

Only then start the stack:

```bash id="lrraf4"
cd /opt/agrupa-cloud
docker compose up -d
```

## Manual recovery if Storage Box is not mounted

Stop Nextcloud:

```bash id="s099bh"
cd /opt/agrupa-cloud
docker compose stop nextcloud
```

If `/mnt/storagebox` is a wrong local directory, move it aside:

```bash id="3tgrsy"
sudo mv /mnt/storagebox /mnt/storagebox.local-wrong-$(date +%Y%m%d-%H%M%S)
sudo mkdir -p /mnt/storagebox
```

Mount the Storage Box:

```bash id="jkzcpx"
sudo sshfs -p 22 u628575@u628575.your-storagebox.de:/ /mnt/storagebox \
  -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3
```

Validate:

```bash id="lr6qag"
findmnt /mnt/storagebox
df -h /mnt/storagebox
sudo ls -la /mnt/storagebox/nextcloud-data | head -40
```

Start Nextcloud:

```bash id="d8sudl"
docker compose up -d nextcloud
docker exec -u www-data -it nextcloud php occ status
```

Expected result:

```txt id="tkelf3"
installed: true
maintenance: false
needsDbUpgrade: false
```