# docs/architecture.md

# AGRUPA Cloud — Architecture

## Overview

AGRUPA Cloud is a self-hosted private cloud environment built with Docker Compose. The system provides file storage, user management, shared team folders, collaborative document editing, database persistence, caching, remote private access, and external storage integration.

The infrastructure is designed around a small-team use case where users need a private alternative to commercial cloud platforms.

## Main services

The project is composed of the following services:

| Service             | Purpose                                                           |
| ------------------- | ----------------------------------------------------------------- |
| Nextcloud           | Main web application for files, users, sharing, and collaboration |
| MariaDB             | Database backend for Nextcloud                                    |
| Redis               | Cache and file locking backend                                    |
| Collabora Online    | Browser-based Office document editing                             |
| WireGuard           | VPN access to the private infrastructure                          |
| Hetzner Storage Box | Remote storage for Nextcloud user data                            |
| Docker Compose      | Service orchestration                                             |

## Logical architecture

```txt
User device
    |
    | WireGuard VPN
    v
Hetzner VPS
    |
    | Docker Compose
    |
    ├── nextcloud
    ├── mariadb
    ├── redis
    ├── collabora
    └── wireguard
          |
          └── /mnt/storagebox
                 |
                 └── Hetzner Storage Box
```

## Storage layout

Nextcloud stores application files, configuration, database data, and user data in separate locations.

| Data type             | Host path                                                     |
| --------------------- | ------------------------------------------------------------- |
| Project files         | `/opt/agrupa-cloud`                                           |
| MariaDB data          | `/mnt/HC_Volume_106259255/mariadb`                            |
| Nextcloud config      | `/var/lib/docker/volumes/agrupa-cloud_nextcloud_config/_data` |
| Nextcloud custom apps | `/var/lib/docker/volumes/agrupa-cloud_nextcloud_apps/_data`   |
| Nextcloud user data   | `/mnt/storagebox/nextcloud-data`                              |

Inside the Nextcloud container, the user data directory is mounted as:

```bash
/var/www/html/data
```

On the host, this maps to:

```bash
/mnt/storagebox/nextcloud-data
```

## Network design

The system is intended to be accessed privately through WireGuard.

Known server addresses:

```txt
Public IP: 116.203.21.170
Private IP used in testing: 10.0.0.2
```

Collabora is exposed on:

```txt
http://10.0.0.2:9980
```

Nextcloud is accessed through the server/VPN address.

## Critical dependency

The most important dependency is the Storage Box mount.

Before starting Nextcloud, this path must be mounted correctly:

```bash
/mnt/storagebox
```

If `/mnt/storagebox` is not mounted, Docker may use a local empty directory instead. This causes Nextcloud to lose access to the real data directory and may produce this error:

```txt
Your data directory is invalid.
Ensure there is a file called ".ncdata" in the root of the data directory.
```

A correct mount should show something similar to:

```txt
u628575@u628575.your-storagebox.de:/  1.0T  204M  1.0T   1% /mnt/storagebox
```

It should not show `/dev/sda1`.
