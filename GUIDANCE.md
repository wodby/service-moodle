# Moodle on Wodby

What Wodby sets up for Moodle on this service.

## Image

The service runs the prebuilt [wodby/moodle](https://github.com/wodby/moodle) image: one Moodle release with its Composer packages on PHP-FPM 8.4, in `/var/www/html`. There is no build and no source repository. New Moodle point releases arrive as new image revisions through service updates; the next deployment upgrades the database.

Moodle's code can't be changed: plugins and themes can't be added, and installing them from Moodle's web interface is turned off (`$CFG->disableupdateautodeploy`), because the image replaces the code on every deployment.

## Settings file

The image's `config.php` takes every setting from the container's environment:

- **Database.** `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME` and `DB_PASSWORD` from the `db` link. `DB_KIND` selects Moodle's driver: `mariadb`, `mysqli` for MySQL, or `pgsql`. Tables use the `mdl_` prefix unless `MOODLE_DB_PREFIX` is set. When the linked server has a CA certificate (`DB_SSL_CA`), Moodle connects over TLS; PostgreSQL also verifies the server against that certificate.
- **Address.** `$CFG->wwwroot` is `MOODLE_WWWROOT` when set, otherwise the app's main route (`WODBY_PRIMARY_URL`, which the container gets through `routeMetadata`). With an `https://` address, `$CFG->sslproxy` is on, because TLS ends before the request reaches Nginx. Moodle answers on that one address only: requests for another domain of the app are redirected to it. A new main route takes effect when the service is redeployed.
- **Data.** `$CFG->dataroot` is `MOODLE_DATAROOT`, `/mnt/files/moodledata` on the shared volume. `$CFG->localcachedir` is `/tmp/moodle-localcache`, inside each container.
- **Sessions.** With a Redis or Valkey service linked as `redis`, sessions are stored there (`REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, key prefix `moodle_sess_`). Otherwise they are files in the data directory.
- **Router.** `$CFG->routerconfigured` is on: the Moodle Nginx service sends requests for unknown paths to `r.php`.
- **Locked settings.** `$CFG->disableupdateautodeploy` and `$CFG->preventexecpath` are on.
- **Mail.** Moodle sends mail with PHP's `mail()`, which the image delivers to the `sendmail` link (`MSMTP_HOST`, `MSMTP_PORT`).

## Data

The `moodledata` volume is mounted at `/mnt/files` and shared by every replica and the cron worker through the linked storage service. Uploaded files, the file-based caches and the temporary directory are kept there. The database holds everything else.

## Cron

The `moodle-cron` derivative runs the image's `moodle-cron`, which keeps Moodle's cron running: each `php admin/cli/cron.php --keep-alive=900` run polls for tasks for 15 minutes and the next run follows. Until Moodle is installed, cron fails and is tried again a minute later. On stop, the current task finishes first.

## Installation and upgrades

Moodle's web installer is not used. After every deployment the service's **Install or upgrade Moodle** action runs the image's `moodle-update`:

- **First deployment:** it installs Moodle into the empty database with `admin/cli/install_database.php`. The administrator is created from the service's settings: `admin-username` (default `admin`, variable `MOODLE_ADMIN_USERNAME`), `admin-password` (required, secret, `MOODLE_ADMIN_PASSWORD`) and `admin-email` (optional, `MOODLE_ADMIN_EMAIL`). The site's full and short names are the app's title and name (`MOODLE_SITE_FULLNAME`, `MOODLE_SITE_SHORTNAME`); both can be changed in Moodle later.
- A password that doesn't meet Moodle's password policy must be changed at the first sign-in. Without an admin email, Moodle asks for it at the first sign-in.
- Without an admin password, the action fails and asks for one; nothing is installed.
- **Later deployments:** it runs `php admin/cli/upgrade.php --non-interactive`. The upgrade turns maintenance mode on while it runs.
- The installer never runs over existing tables: if the database can't be read, the action fails instead.
- Changing the admin settings after the first deployment does not change the account; change it in Moodle.

## Actions

- **Purge all caches** runs `php admin/cli/purge_caches.php`.

## PHP settings

The image sets `max_input_vars` to 5000, as Moodle requires, limits uploads and form posts to 256 MB, matching the Nginx service, and turns `zend.exception_ignore_args` on, so stack traces don't show argument values. In production environments PHP doesn't display errors.

## Limits

- Moodle 5.3 needs MariaDB 11.4 or later, MySQL 8.4 or later or PostgreSQL 17 or later.
- It has no development workspace.

## Check the result

- **Site administration > Server > Environment** shows Moodle's own checks.
- `php admin/cli/cfg.php --name=version` prints the installed version; it fails before installation.
- The deployment's post-deploy log shows the install or upgrade output.
- `php admin/cli/cron.php --list` in the cron worker lists the tasks running now.
