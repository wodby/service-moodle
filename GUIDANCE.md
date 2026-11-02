# Moodle on Wodby

What Wodby sets up for Moodle on this service, in addition to the PHP service it is based on.

## Settings file

Moodle reads `config.php` from the repository root, next to `public/`. When the repository has no `config.php`, the image build adds one that takes every setting from the container's environment, so the same image runs in every environment:

- **Database.** `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME` and `DB_PASSWORD` from the `db` link. `DB_KIND` selects Moodle's driver: `mariadb`, `mysqli` for MySQL, or `pgsql`. Tables use the `mdl_` prefix unless `MOODLE_DB_PREFIX` is set. When the linked server has a CA certificate (`DB_SSL_CA`), Moodle connects over TLS; PostgreSQL also verifies the server against that certificate.
- **Address.** `$CFG->wwwroot` is `MOODLE_WWWROOT` when set, otherwise the app's main route (`WODBY_PRIMARY_URL`). With an `https://` address, `$CFG->sslproxy` is on, because TLS ends before the request reaches Nginx. Moodle answers on that one address only: requests for another domain of the app are redirected to it. A new main route takes effect when the service is redeployed.
- **Data.** `$CFG->dataroot` is `MOODLE_DATAROOT`, `/mnt/files/moodledata` on the shared volume. `$CFG->localcachedir` is `/tmp/moodle-localcache`, inside each container.
- **Sessions.** With a Redis or Valkey service linked as `redis`, sessions are stored there (`REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, key prefix `moodle_sess_`). Otherwise they are files in the data directory.
- **Router.** `$CFG->routerconfigured` is on: the Moodle Nginx service sends requests for unknown paths to `r.php`.
- **Locked settings.** `$CFG->disableupdateautodeploy` and `$CFG->preventexecpath` are on. Plugins are added to the repository, not installed from the web interface, because the code is part of the image.

A repository that has its own `config.php` keeps it, and none of the above applies. Do not commit credentials; read them from the variables above.

## Build

The image is built from the repository with this service's Dockerfile: the code is copied to `/var/www/html`, and the settings file above is added when missing. The pipeline of the Vanilla Moodle boilerplate runs `composer install --no-dev --classmap-authoritative` first; Moodle's router and `bin/moodle` need those packages.

## Data

The `moodledata` volume is mounted at `/mnt/files` and shared by every replica and the cron worker through the linked storage service. Uploaded files, the file-based caches and the temporary directory are kept there. The database holds everything else.

## Cron

The `moodle-cron` derivative runs Moodle's cron continuously: each `php admin/cli/cron.php --keep-alive=900` run polls for tasks for 15 minutes and the next run follows. Until Moodle is installed, cron fails and is tried again a minute later. On stop, the current task finishes first.

## Installation and upgrades

- Moodle is installed from the web: open the site, accept the licence, then create the administrator account. Until that is done, anyone who opens the site can do it.
- After every deployment the service runs `php admin/cli/upgrade.php --non-interactive` once Moodle is installed. The upgrade turns maintenance mode on while it runs.

## Actions

- **Purge all caches** runs `php admin/cli/purge_caches.php`.

## PHP settings

`max_input_vars` is 5000, as Moodle requires. Uploads and form posts are limited to 256 MB, matching the Nginx service. In production environments `zend.exception_ignore_args` is on.

## Limits

- Moodle 5.3 supports PHP 8.3 and 8.4, MariaDB 11.4 or later, MySQL 8.4 or later and PostgreSQL 17 or later.
- It has no development workspace.

## Check the result

- **Site administration > Server > Environment** shows Moodle's own checks.
- `php admin/cli/cfg.php --name=version` prints the installed version; it fails before installation.
- `php admin/cli/cron.php --list` in the cron worker lists the tasks running now.
