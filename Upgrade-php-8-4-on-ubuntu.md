# Upgrade PHP to 8.4 on Ubuntu

This guide explains how to upgrade PHP to **PHP 8.4** on Ubuntu.

It is written for real server usage, especially Laravel projects using Apache or Nginx.

---

## What This Guide Covers

- Check current PHP version
- Add PHP repository
- Install PHP 8.4
- Install common Laravel extensions
- Set PHP 8.4 as default CLI PHP
- Configure Apache or Nginx
- Fix Redis / Laravel Horizon issue after PHP upgrade
- Verify everything is working

---

## Important Placeholders

Change these values according to your own server:

| Placeholder                                   | Replace With                            |
| --------------------------------------------- | --------------------------------------- |
| `php_old_version_goes_here`                   | Your old PHP version, for example `8.3` |
| `php_new_version_goes_here`                   | Your new PHP version, for example `8.4` |
| `/path/to/your/project`                       | Your Laravel/project path               |
| `/etc/nginx/sites-available/your-site-config` | Your actual Nginx site config file      |

Example:

```txt
php_old_version_goes_here = 8.3
php_new_version_goes_here = 8.4
```

---

# 1. Check Current PHP Version

Run:

```bash
php -v
```

This shows the PHP version currently used in the terminal.

You can also check installed PHP packages:

```bash
dpkg -l | grep php
```

Optional: save your current PHP package list before upgrading:

```bash
dpkg -l | grep php | tee php-packages-before-upgrade.txt
```

This is helpful because after upgrading, you can check which old extensions were installed and install the same extensions for PHP 8.4.

---

# 2. Update Ubuntu Package List

Run:

```bash
sudo apt update
```

This refreshes Ubuntu package information.

---

# 3. Install Required Helper Packages

Run:

```bash
sudo apt install -y software-properties-common ca-certificates apt-transport-https
```

These packages help Ubuntu add and use external repositories safely.

---

# 4. Add PHP Repository

Run:

```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
```

This repository provides newer PHP versions such as PHP 8.4 for Ubuntu.

---

# 5. Install PHP 8.4 and Common Laravel Extensions

Run:

```bash
sudo apt install -y \
php8.4 \
php8.4-cli \
php8.4-fpm \
php8.4-common \
php8.4-mbstring \
php8.4-xml \
php8.4-curl \
php8.4-zip \
php8.4-bcmath \
php8.4-mysql \
php8.4-pgsql \
php8.4-gd \
php8.4-intl
```

These are common extensions used by Laravel and most PHP projects.

---

# 6. Optional Extensions

Install these only if your project needs them.

## Redis / Laravel Horizon

If your project uses Redis, queues, cache, sessions, or Laravel Horizon, install:

```bash
sudo apt install -y php8.4-redis
```

## Imagick

If your project processes images, install:

```bash
sudo apt install -y php8.4-imagick
```

## SOAP

If your project uses SOAP APIs, install:

```bash
sudo apt install -y php8.4-soap
```

## SQLite

If your project uses SQLite, install:

```bash
sudo apt install -y php8.4-sqlite3
```

---

# 7. Set PHP 8.4 as Default Terminal PHP

Run:

```bash
sudo update-alternatives --set php /usr/bin/php8.4
```

Verify:

```bash
php -v
```

Expected result:

```txt
PHP 8.4.x
```

If you want to select PHP version manually, run:

```bash
sudo update-alternatives --config php
```

Then select PHP 8.4 from the list.

---

# 8. Configure Web Server

Use only the section that matches your server.

---

## Option A: Apache Server

If your server uses Apache with PHP module, first install the Apache PHP 8.4 module:

```bash
sudo apt install -y libapache2-mod-php8.4
```

Disable old PHP versions.

Example:

```bash
sudo a2dismod php8.2 php8.3
```

If your old version is different, replace it:

```bash
sudo a2dismod php_old_version_goes_here
```

Enable PHP 8.4:

```bash
sudo a2enmod php8.4
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

Check Apache status:

```bash
sudo systemctl status apache2
```

---

## Option B: Nginx Server

Nginx usually uses PHP-FPM.

Open your Nginx site config:

```bash
sudo nano /etc/nginx/sites-available/your-site-config
```

Example:

```bash
sudo nano /etc/nginx/sites-available/default
```

Find the old PHP-FPM socket:

```nginx
fastcgi_pass unix:/run/php/php8.3-fpm.sock;
```

Change it to:

```nginx
fastcgi_pass unix:/run/php/php8.4-fpm.sock;
```

## What to Change

Change only the PHP version number in the socket path.

Before:

```txt
php8.3-fpm.sock
```

After:

```txt
php8.4-fpm.sock
```

Now test Nginx config:

```bash
sudo nginx -t
```

If the test is successful, restart PHP-FPM and Nginx:

```bash
sudo systemctl restart php8.4-fpm
sudo systemctl restart nginx
```

Check services:

```bash
sudo systemctl status php8.4-fpm
sudo systemctl status nginx
```

---

# 9. Clear Laravel Cache

Go to your project:

```bash
cd /path/to/your/project
```

Example:

```bash
cd /var/www/your-project-folder
```

Clear Laravel cache:

```bash
php artisan optimize:clear
```

This clears cached config, routes, views, and other optimized files.

---

# 10. Restart Laravel Horizon / Queue

If your project uses Laravel Horizon, run:

```bash
php artisan horizon:terminate
```

This safely stops Horizon so Supervisor can start it again with the new PHP version.

Check Horizon status:

```bash
php artisan horizon:status
```

If you see:

```txt
Horizon is inactive.
```

It means Horizon is currently stopped. This can happen after `horizon:terminate`.

If you are using Supervisor, restart it:

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl restart all
```

Then check again:

```bash
php artisan horizon:status
```

---

# 11. Fix Redis Error After PHP Upgrade

After upgrading PHP, you may see this Laravel error:

```txt
Class "Redis" not found
```

This usually means Redis extension was installed for the old PHP version, for example PHP 8.3, but not for PHP 8.4.

Fix:

```bash
sudo apt install -y php8.4-redis
```

Check Redis extension:

```bash
php -m | grep redis
```

Clear Laravel cache:

```bash
php artisan optimize:clear
```

Restart PHP-FPM:

```bash
sudo systemctl restart php8.4-fpm
```

If using Horizon:

```bash
php artisan horizon:terminate
sudo supervisorctl restart all
```

---

# 12. Verify PHP Extensions

Check all loaded PHP extensions:

```bash
php -m
```

Check specific extensions:

```bash
php -m | grep redis
php -m | grep pdo
php -m | grep pgsql
php -m | grep mysql
php -m | grep mbstring
php -m | grep curl
```

---

# 13. Final Verification

Check PHP CLI version:

```bash
php -v
```

Check PHP-FPM:

```bash
sudo systemctl status php8.4-fpm
```

Check web server.

For Apache:

```bash
sudo systemctl status apache2
```

For Nginx:

```bash
sudo systemctl status nginx
```

Check Laravel:

```bash
php artisan about
```

---

# Common Problems and Fixes

## Problem 1: `Class "Redis" not found`

Reason:

Redis extension is missing for PHP 8.4.

Fix:

```bash
sudo apt install -y php8.4-redis
php -m | grep redis
php artisan optimize:clear
sudo systemctl restart php8.4-fpm
```

---

## Problem 2: Nginx Still Uses Old PHP Version

Reason:

Nginx site config still points to old PHP-FPM socket.

Find old socket references:

```bash
grep -R "php8." /etc/nginx/sites-available/
```

Change:

```txt
php8.3-fpm.sock
```

to:

```txt
php8.4-fpm.sock
```

Then run:

```bash
sudo nginx -t
sudo systemctl restart php8.4-fpm
sudo systemctl restart nginx
```

---

## Problem 3: `a2enmod php8.4` Not Found

Reason:

Apache PHP 8.4 module is not installed.

Fix:

```bash
sudo apt install -y libapache2-mod-php8.4
sudo a2enmod php8.4
sudo systemctl restart apache2
```

---

## Problem 4: Composer or Laravel Still Shows Old PHP

Reason:

Terminal PHP default may still point to old version.

Fix:

```bash
sudo update-alternatives --config php
php -v
```

Select PHP 8.4.

---

## Problem 5: Horizon is Inactive

Reason:

After running:

```bash
php artisan horizon:terminate
```

Horizon stops safely and waits for Supervisor/process manager to restart it.

Fix:

```bash
sudo supervisorctl restart all
php artisan horizon:status
```

---

# Quick Command List

For Nginx + Laravel server:

```bash
php -v

sudo apt update
sudo apt install -y software-properties-common ca-certificates apt-transport-https

sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

sudo apt install -y \
php8.4 \
php8.4-cli \
php8.4-fpm \
php8.4-common \
php8.4-mbstring \
php8.4-xml \
php8.4-curl \
php8.4-zip \
php8.4-bcmath \
php8.4-mysql \
php8.4-pgsql \
php8.4-gd \
php8.4-intl \
php8.4-redis

sudo update-alternatives --set php /usr/bin/php8.4

php -v
php -m | grep redis

sudo nginx -t
sudo systemctl restart php8.4-fpm
sudo systemctl restart nginx

cd /path/to/your/project
php artisan optimize:clear
php artisan horizon:terminate

sudo supervisorctl restart all
php artisan horizon:status
```

---

# Notes From My Experience

After upgrading from PHP 8.3 to PHP 8.4, Laravel Horizon showed Redis-related errors because Redis was installed for the old PHP version only.

The fix was:

```bash
sudo apt install -y php8.4-redis
```

After installing the correct Redis extension for PHP 8.4, Laravel commands worked again.

Also, after running:

```bash
php artisan horizon:terminate
```

Horizon may show:

```txt
Horizon is inactive.
```

This is not always a problem. It usually means Horizon was stopped and needs to be restarted by Supervisor or your process manager.

---

# References

- PHP 8.4 packages and extensions:
  https://php.watch/articles/php-84-install-upgrade-guide-debian-ubuntu

- Ubuntu `update-alternatives`:
  https://manpages.ubuntu.com/manpages/jammy/man1/update-alternatives.1.html

- Laravel Horizon:
  https://laravel.com/docs/horizon

- PHP-FPM with Apache/Nginx:
  https://www.linode.com/docs/guides/install-php-8-for-apache-and-nginx-on-ubuntu/

---

# Summary

To upgrade PHP safely:

1. Check your current PHP version.
2. Install PHP 8.4 and required extensions.
3. Set PHP 8.4 as default CLI PHP.
4. Update Apache or Nginx configuration.
5. Restart PHP-FPM/web server.
6. Clear Laravel cache.
7. Restart Horizon/queue workers.
8. Verify extensions like Redis if your project depends on them.
