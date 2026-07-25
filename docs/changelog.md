# docs/changelog.md

# AGRUPA Cloud — Changelog

## 2026-07-23 — Recovery and restoration

### Added

* Restored old Nextcloud configuration from backup.
* Restored old MariaDB data from backup.
* Reconnected Hetzner Storage Box through SSHFS.
* Validated Nextcloud installation with `occ status`.
* Confirmed Nextcloud version `31.0.14`.
* Confirmed `maintenance: false`.
* Confirmed `needsDbUpgrade: false`.

### Fixed

* Fixed issue where Nextcloud asked to create a new admin account.
* Fixed missing/incomplete `config.php`.
* Fixed empty active MariaDB database.
* Fixed invalid data directory caused by unmounted Storage Box.
* Fixed access to old user data stored in the Storage Box.

### Confirmed

* Storage Box mounted at:

```txt id="tfe43x"
/mnt/storagebox
```

* Nextcloud data directory available at:

```txt id="q6hqwl"
/mnt/storagebox/nextcloud-data
```

* Existing data directories found:

```txt id="kd0kd0"
AGRUPA-admin
Dina
Izequiel
Jose Filipe
__groupfolders
appdata_ochhd2e8gwyw
```

* Collabora endpoint previously tested successfully at:

```txt id="l4akjf"
http://10.0.0.2:9980/hosting/discovery
```

## Next planned changes

* Make Storage Box mount persistent.
* Ensure Docker starts only after Storage Box is mounted.
* Configure HTTPS/domain.
* Finalize Nextcloud Office/Collabora browser editing.
* Automate backups.
* Add monitoring and alerting.
