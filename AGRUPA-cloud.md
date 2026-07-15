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

graph TB
    subgraph VPS["VPS"]
        subgraph Containers["Docker Containers"]
            NC["Nextcloud"]
            WG["Wireguard VPN"]
            MDB["MariaDB"]
            RD["Redis"]
            CO["Collabora Online"]
            CD["CoreDNS"]
        end
        subgraph Storage["Persistent Storage"]
            VOL["MariaDB Volume"]
        end
    end
    
    EMP["Employee"]
    INT["Internet"]
    
    EMP -->|VPN Connection| WG
    WG -->|VPN Active| NC
    NC -->|Query Data| MDB
    MDB -->|Read/Write| VOL
    NC -->|Cache Layer| RD
    NC -->|Document Editing| CO
    WG -->|DNS Resolution| CD
    CD -->|DNS Queries| INT
    EMP -->|No VPN| INT
    INT -.->|Blocked| NC
    
    classDef vpnContainer stroke:#e879f9,fill:#fdf4ff
    classDef dataContainer stroke:#2dd4bf,fill:#f0fdfa
    classDef appContainer stroke:#818cf8,fill:#eef2ff
    classDef storage stroke:#4ade80,fill:#f0fdf4
    classDef external stroke:#fb923c,fill:#fff7ed
    classDef vpsBox stroke:#a78bfa,fill:#f5f3ff
    
    class WG vpnContainer
    class MDB,RD dataContainer
    class NC,CO,CD appContainer
    class VOL storage
    class EMP,INT external
    class VPS,Containers vpsBox
