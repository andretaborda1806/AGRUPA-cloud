# docs/known-issues.md

# AGRUPA Cloud — Known Issues

## 1. Storage Box mount is manual

The Storage Box currently needs to be mounted manually with SSHFS.

Command:

```bash id="5n7at5"
sudo sshfs -p 22 u628575@u628575.your-storagebox.de:/ /mnt/storagebox \
  -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3
```

Risk:

```txt id="ujzwgm"
After a reboot, Nextcloud may start before /mnt/storagebox is mounted.
```

Impact:

```txt id="2790gw"
Nextcloud may fail with invalid data directory.
```

Required improvement:

```txt id="sz249h"
Create a persistent systemd mount or fstab entry and ensure Docker starts after the mount.
```

## 2. HTTP is still being used internally

Current access has been tested through:

```txt id="w752tv"
http://10.0.0.2
```

Recommended improvement:

```txt id="5nwl2f"
Configure HTTPS with a proper domain or internal trusted certificate.
```

## 3. Collabora depends on correct URL

Collabora was tested through:

```txt id="4j28fp"
http://10.0.0.2:9980/hosting/discovery
```

If the user accesses Nextcloud through a different hostname, Collabora configuration may need to be adjusted.

## 4. Browser-based Office editing requires Nextcloud Office

If documents download instead of opening in the browser, the `richdocuments` app may not be installed or configured.

Fix:

```bash id="u2yuv1"
docker exec -u www-data -it nextcloud php occ app:install richdocuments
docker exec -u www-data -it nextcloud php occ app:enable richdocuments
```

## 5. Backups exist but automation should be improved

Backups were found during recovery, but the backup process should be automated and tested regularly.

Recommended:

```txt id="76y3yd"
scheduled database dumps
scheduled config backups
separate backup for user data
restore test procedure
```

## 6. Service startup order matters

Nextcloud depends on:

```txt id="rdfqq4"
Storage Box
MariaDB
Redis
```

If these are not available, Nextcloud may fail.

---