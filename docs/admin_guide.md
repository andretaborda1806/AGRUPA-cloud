

# AGRUPA Cloud — Administrator Guide

## Purpose

This document explains how to manage users, groups, passwords, team folders, and Office integration in Nextcloud.

## Access

Admin access is done through the Nextcloud web interface.

Current internal access:

```txt id="lw0rgr"
http://10.0.0.2
```

The server is administered through SSH:

```bash id="mc8nbt"
ssh andre@116.203.21.170
```

Project directory:

```bash id="6nxlbw"
cd /opt/agrupa-cloud
```

## Check system status

```bash id="u5m5mp"
docker compose ps
docker exec -u www-data -it nextcloud php occ status
```

Expected:

```txt id="hcwyvk"
installed: true
maintenance: false
needsDbUpgrade: false
```

## List users

```bash id="w7tsvl"
docker exec -u www-data -it nextcloud php occ user:list
```

## Reset user password

Passwords cannot be recovered in plain text. They must be reset.

```bash id="gw23hy"
docker exec -u www-data -it nextcloud php occ user:resetpassword USERNAME
```

Example:

```bash id="0by406"
docker exec -u www-data -it nextcloud php occ user:resetpassword Dina
```

For usernames with spaces:

```bash id="eothsb"
docker exec -u www-data -it nextcloud php occ user:resetpassword "Jose Filipe"
```

## Recommended password reset policy

1. Generate a temporary password.
2. Share it through a secure channel.
3. Ask the user to change it after login.
4. Avoid keeping user passwords.

## Groups

Groups should be used to control access to shared folders.

Recommended group:

```txt id="cusqvd"
AGRUPA
```

Users can be managed through:

```txt id="6zf96g"
Admin settings → Users
```

## Team Folders

Team Folders are used for shared folders managed by the administrator.

Enable app:

```bash id="j2frah"
docker exec -u www-data -it nextcloud php occ app:install groupfolders
docker exec -u www-data -it nextcloud php occ app:enable groupfolders
```

Manage through:

```txt id="tjmu5w"
Admin settings → Team folders
```

Recommended permissions for a shared editable folder:

```txt id="jq8mm4"
Read
Write
Create
Delete
Share
```

## Nextcloud Office

To open Office files inside the browser, enable:

```txt id="416454"
richdocuments
```

Commands:

```bash id="4rcaei"
docker exec -u www-data -it nextcloud php occ app:install richdocuments
docker exec -u www-data -it nextcloud php occ app:enable richdocuments
```

Configure:

```txt id="n95hc5"
Admin settings → Office → Use your own server
```

Collabora URL:

```txt id="iddk0b"
http://10.0.0.2:9980
```

## Maintenance mode

Enable maintenance mode:

```bash id="fxiirx"
docker exec -u www-data -it nextcloud php occ maintenance:mode --on
```

Disable maintenance mode:

```bash id="foiqjf"
docker exec -u www-data -it nextcloud php occ maintenance:mode --off
```

## Repair

```bash id="6s1yst"
docker exec -u www-data -it nextcloud php occ maintenance:repair
```

## File scan

```bash id="6s5gkk"
docker exec -u www-data -it nextcloud php occ files:scan --all
```

---

