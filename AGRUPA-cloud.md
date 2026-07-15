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
  theme: neutral
---
flowchart TB
 subgraph Containers["Docker Containers"]
        NC(["Nextcloud"])
        WG(["Wireguard VPN"])
        MDB["MariaDB"]
        RD(["Redis"])
        CO(["Collabora Online"])
        CD(["CoreDNS"])
  end
 subgraph Storage["Persistent Storage"]
        VOL["MariaDB Volume"]
  end
 subgraph VPS["VPS"]
        Containers
        Storage
  end
    EMP(["Employee"]) -- VPN Connection --> WG
    WG -- VPN Active --> NC
    NC -- Query Data --> MDB
    MDB -- Read/Write --> VOL
    NC -- Cache Layer --> RD
    NC -- Document Editing --> CO
    WG -- DNS Resolution --> CD
    CD -- DNS Queries --> INT(["Internet"])
    EMP -- No VPN --> INT
    INT -. Blocked .-> NC

    MDB@{ shape: rounded}
    style VOL fill:#ffffff
    style Containers fill:#fcfcfc,stroke:#fcfcfc
    style Storage fill:#FFE0B2,stroke:#FFE0B2
    style VPS fill:#C8E6C9,stroke:#C8E6C9
```
