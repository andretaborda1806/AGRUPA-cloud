# Architecture

## Overview

This project provides a private cloud storage platform for a small company.
This project provides a private cloud storage platform for a small company.

## Goals

The goal is to offer a Google Drive-like experience while keeping access restricted to authorized users through a private Tailscale network.

## System Components

### Nextcloud

Nextcloud is the main component of this project.

It provides:

* Main user interface
* User login
* File upload and download

Inside the Docker network, Nextcloud only communicates with the required internal services and is accessed through Tailscale.

It is not publicly exposed.

### Tailscale

If Nextcloud is the house, Tailscale is the door.

Tailscale manages access by allowing only authorized devices to reach the platform.

It uses Tailscale Serve to proxy HTTPS traffic to the internal Nextcloud container.

### MariaDB

MariaDB is the backend database for Nextcloud.

It stores:

* Users
* File metadata
* Permissions

It listens on the internal port `3306`.

It is not publicly exposed.

### Redis

Redis stores Nextcloud cache and handles file locking in order to improve performance.

It is not publicly exposed.

## Network Architecture

All containers are connected to a private Docker network.

## Access Flow

<table>
    <td>User opens the Tailscale app</td>
  <tr>
    <td align="center">↓</td>
  </tr>
  <tr>
    <td>User connects to the tailnet</td>
  </tr>
  <tr>
    <td align="center">↓</td>
  </tr>
  <tr>
    <td>User opens the Tailscale Serve URL</td>
  </tr>
  <tr>
    <td align="center">↓</td>
  </tr>
  <tr>
    <td>Tailscale proxies the request to Nextcloud</td>
  </tr>
  <tr>
    <td align="center">↓</td>
  </tr>
  <tr>
    <td>User logs in with a Nextcloud account</td>
  </tr>
</table>

## Docker Services

All services are connected using a private Docker network.

## Storage Architecture

<table>
  <tr>
    <td>User uploads a file</td>
  </tr>
  <tr>
    <td align="center">↓</td>
  </tr>
  <tr>
    <td>Nextcloud writes to <code>/var/www/html/data</code></td>
  </tr>
  <tr>
    <td align="center">↓</td>
  </tr>
  <tr>
    <td>Docker maps the directory to <code>/mnt/agrupa/nextcloud-data</code></td>
  </tr>
  <tr>
    <td align="center">↓</td>
  </tr>
  <tr>
    <td>The file is stored in block storage</td>
  </tr>
</table>

## Security Boundaries

Tailscale controls network-level access.

This means that users outside the tailnet cannot reach the platform, while users inside the tailnet still need valid Nextcloud credentials.

## Design Decisions

Docker was chosen to simplify deployment and make the stack reproducible.

Tailscale was chosen to avoid exposing the platform directly to the public internet.

Block storage was chosen because it is expandable and cost-effective. It also keeps user files separate from the VPS root storage.

Redis was chosen to improve caching and file locking.

## Limitations / Future Improvements

The current free use of Tailscale is better suited for small-scale usage due to cost constraints.

The dependence on a service like Tailscale requires every user to install the app.

A complete backup strategy is still needed for both user files and the MariaDB database.
