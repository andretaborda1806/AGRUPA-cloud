# Agrupa Cloud

## Description

This project provides a private cloud storage solution for a small company that needs to store confidential information securely, reduce the cost and limitations of expanding local storage, and make file access easier for company partners.

The platform uses Nextcloud to provide a Google Drive-like experience, Tailscale to restrict access to authorized devices, and block storage to keep user files separate from the VPS root disk. The system also provides a foundation for future backup and restore procedures.

## Features

- Private cloud storage platform
- File upload and download through Nextcloud
- User authentication and account management
- Access restricted through Tailscale private network
- No public exposure of Nextcloud, MariaDB or Redis
- User files stored on mounted block storage
- MariaDB database for Nextcloud metadata
- Redis support for caching and file locking
- Docker Compose deployment
- Foundation for backup and restore procedures

## Technologies

- Docker
- Docker Compose
- Nextcloud
- MariaDB
- Redis
- Tailscale
- Tailscale Serve
- Linux VPS (Virtual Private Server)
- Block Storage

## Architecture Overview

## Requirements

## Quick Start

## Documentation

## Security Notice