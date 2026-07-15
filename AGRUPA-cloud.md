# AGRUPA-cloud
## General Overview

A private, docker-based collaboration platform for a small company. The platform is intended to keep internal services protected from direct public access by requiring users to connect through a VPN.

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

    classDef vpnContainer stroke:#e879f9,fill:#fdf4ff
    classDef dataContainer stroke:#2dd4bf,fill:#f0fdfa
    classDef appContainer stroke:#818cf8,fill:#eef2ff
    classDef storage stroke:#4ade80,fill:#f0fdf4
    classDef external stroke:#fb923c,fill:#fff7ed
    classDef externalStorage stroke:#38bdf8,fill:#f0f9ff
    class WG vpnContainer
    class MDB,RD dataContainer
    class NC,CO appContainer
    class VOL storage
    class NCDATA externalStorage
    class EMP,INT external
```
