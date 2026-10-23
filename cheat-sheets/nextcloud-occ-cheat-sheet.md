# Nextcloud occ cheat sheet

Field-tested occ commands for running Nextcloud – searchable and copyable in one click. Run them from the Nextcloud directory, e.g. /var/www/nextcloud.

## Maintenance & updates

- Show version, install state and maintenance mode

  ```bash
  sudo -u www-data php occ status
  ```
- Enable maintenance mode (locks out users)

  ```bash
  sudo -u www-data php occ maintenance:mode --on
  ```
- Disable maintenance mode

  ```bash
  sudo -u www-data php occ maintenance:mode --off
  ```
- Finish an upgrade after replacing the code files

  ```bash
  sudo -u www-data php occ upgrade
  ```
- Run repair steps (mimetypes, caches and more)

  ```bash
  sudo -u www-data php occ maintenance:repair
  ```
- Verify core file integrity against signatures

  ```bash
  sudo -u www-data php occ integrity:check-core
  ```

## Files, trash bin & versions

- Rescan the file system for all users

  ```bash
  sudo -u www-data php occ files:scan --all
  ```
- Rescan files of a single user

  ```bash
  sudo -u www-data php occ files:scan USER
  ```
- Rescan only one folder (faster)

  ```bash
  sudo -u www-data php occ files:scan --path="/USER/files/FOLDER"
  ```
- Rescan the appdata folder (e.g. previews)

  ```bash
  sudo -u www-data php occ files:scan-app-data
  ```
- Remove orphaned entries from the file cache

  ```bash
  sudo -u www-data php occ files:cleanup
  ```
- Permanently empty the trash bin of all users (**careful**)

  ```bash
  sudo -u www-data php occ trashbin:cleanup --all-users
  ```
- Delete trash bin files past the retention policy (**careful**)

  ```bash
  sudo -u www-data php occ trashbin:expire
  ```
- Delete all file versions of all users (**careful**)

  ```bash
  sudo -u www-data php occ versions:cleanup
  ```

## Users & groups

- List all users

  ```bash
  sudo -u www-data php occ user:list
  ```
- Show user details (quota, groups, last login)

  ```bash
  sudo -u www-data php occ user:info USER
  ```
- Create a user and add to a group (prompts for password)

  ```bash
  sudo -u www-data php occ user:add --display-name="Max Mustermann" --group="staff" USER
  ```
- Reset a user's password

  ```bash
  sudo -u www-data php occ user:resetpassword USER
  ```
- Disable a user (data is kept)

  ```bash
  sudo -u www-data php occ user:disable USER
  ```
- Delete a user including all files (**careful**)

  ```bash
  sudo -u www-data php occ user:delete USER
  ```
- Show a user's last login

  ```bash
  sudo -u www-data php occ user:lastseen USER
  ```
- Add a user to a group

  ```bash
  sudo -u www-data php occ group:adduser GROUP USER
  ```

## Apps

- List enabled and disabled apps with versions

  ```bash
  sudo -u www-data php occ app:list
  ```
- Enable an app

  ```bash
  sudo -u www-data php occ app:enable APP
  ```
- Disable an app (e.g. after a broken update)

  ```bash
  sudo -u www-data php occ app:disable APP
  ```
- Update all apps to the latest version

  ```bash
  sudo -u www-data php occ app:update --all
  ```

## Database, cron & config

- Add missing database indices (admin overview warning)

  ```bash
  sudo -u www-data php occ db:add-missing-indices
  ```
- Switch background jobs to system cron

  ```bash
  sudo -u www-data php occ background:cron
  ```
- Show trusted domains

  ```bash
  sudo -u www-data php occ config:system:get trusted_domains
  ```
- Add a domain as index 2 to trusted_domains

  ```bash
  sudo -u www-data php occ config:system:set trusted_domains 2 --value="cloud.example.com"
  ```
- Set log level to warning (0=debug … 4=fatal)

  ```bash
  sudo -u www-data php occ config:system:set loglevel --value=2 --type=integer
  ```

## Security & logs

- Show server-side encryption status

  ```bash
  sudo -u www-data php occ encryption:status
  ```
- Show 2FA state and providers of a user

  ```bash
  sudo -u www-data php occ twofactorauth:state USER
  ```
- Disable TOTP 2FA for a user (lost device) (**careful**)

  ```bash
  sudo -u www-data php occ twofactorauth:disable USER totp
  ```
- Show the last 100 log entries (logreader app)

  ```bash
  sudo -u www-data php occ log:tail 100
  ```

---
More detail, related guides and tools: **[daniele-michelin.com/en/cheat-sheets/nextcloud-occ-cheat-sheet/](https://daniele-michelin.com/en/cheat-sheets/nextcloud-occ-cheat-sheet/)**
