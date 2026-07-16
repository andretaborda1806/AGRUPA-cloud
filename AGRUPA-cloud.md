# AGRUPA-cloud
## General Overview

A private, docker-based collaboration platform for a small company. The platform is intended to keep internal services protected from direct public access by requiring users to connect through a VPN.

## Features

- Private file storage
- Secure remote access
- Online document editing
- Private DNS
- Persistent database
- Caching and file locking

## Technologies

* Docker
* Nextcloud
* MariaDB
* Redis
* Wireguard
* CoreDNS
* Collabora ONLINE

## Architecture

```mermaid
---
config:
  layout: elk
---
flowchart TB
    subgraph VPS["VPS"]
        subgraph Containers["Docker Containers"]
            WG["Wireguard VPN"]
            NC["Nextcloud"]
            MDB["MariaDB"]
            RD["Redis"]
            CO["Collabora Online"]
        end
    end

    subgraph ExternalStorage["External Storage Box"]
        NCDATA["Nextcloud Data"]
    end

    subgraph Storage["Persistent Storage (local)"]
        VOL["MariaDB Volume"]
    end

    EMP["Employee"]
    INT["Internet"]

    EMP -->|Open VPN Tunnel| WG
    WG -->|Tunnel Established| NC
    NC -->|Authenticate & Authorize| MDB
    NC -->|Store Files| NCDATA
    MDB -->|Persistent Data| VOL
    NC -->|Cache Data| RD
    NC -->|Use Collabora| CO
    EMP -.->|Tries without VPN| INT
    INT -.->|Access Blocked| NC

    %% Color definitions optimized for GitHub dark mode
    classDef vpnContainer stroke:#c084fc,fill:#4c1d95,color:#ffffff
    classDef dataContainer stroke:#2dd4bf,fill:#134e4a,color:#ffffff
    classDef appContainer stroke:#818cf8,fill:#1e1b4b,color:#ffffff
    classDef storage stroke:#4ade80,fill:#14532d,color:#ffffff
    classDef external stroke:#fb923c,fill:#7c2d12,color:#ffffff
    classDef externalStorage stroke:#38bdf8,fill:#0c4a6e,color:#ffffff

    class WG vpnContainer
    class MDB,RD dataContainer
    class NC,CO appContainer
    class VOL storage
    class NCDATA externalStorage
    class EMP,INT external
```
## Requirements

- Linux server
- Docker
- Docker Compose

## 
