
# AGRUPA Cloud — Future Improvements

## 1. Persistent Storage Box mount

Create a systemd mount or service for SSHFS.

Goal:

```txt id="pxk53i"
/mnt/storagebox is mounted automatically after reboot.
```

The Docker stack should only start after the Storage Box is available.

## 2. HTTPS and domain configuration

Configure a domain and HTTPS.

Possible improvements:

```txt id="q3qhbg"
reverse proxy
TLS certificate
automatic certificate renewal
trusted_domains update in Nextcloud
overwrite.cli.url update
```

## 3. Backup automation

Automate backups for:

```txt id="8w4g5k"
Nextcloud config
MariaDB dump
project files
secrets
Storage Box data
```

## 4. Monitoring

Add basic monitoring for:

```txt id="m8sx9k"
container status
disk usage
Storage Box mount status
Nextcloud status
MariaDB health
Collabora availability
```

## 5. Alerts

Configure alerts for:

```txt id="fjwdq5"
Storage Box unmounted
disk almost full
container down
backup failed
Nextcloud maintenance mode enabled
```

## 6. Security hardening

Recommended:

```txt id="2z5ioh"
enable 2FA for admins
restrict SSH access
use SSH keys
review firewall rules
avoid public exposure of internal services
protect .env and secrets
```

## 7. Documented restore test

Perform and document a full restore test.

A full restore should validate:

```txt id="ek08lu"
config restoration
database restoration
Storage Box mount
Nextcloud login
file access
Office editing
```

## 8. CI/CD for configuration

Possible future improvement:

```txt id="t1fpik"
Store sanitized Docker Compose configuration in Git.
Use private secret storage for sensitive values.
Track infrastructure changes through commits.
```
