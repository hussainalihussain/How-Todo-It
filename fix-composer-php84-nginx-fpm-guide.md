# Fix Composer PHP Version Error on Server

## Problem

Composer shows this error:

```text
Composer detected issues in your platform:
Your Composer dependencies require a PHP version ">= 8.4.0".
```

In `composer.json`, the PHP requirement is:

```json
"php": "^8.4"
```

This is correct.

`^8.4` means:

```text
PHP >= 8.4 and < 9.0
```

So if PHP 8.4 is installed, Composer should not complain.

---

## Real Cause

The server had PHP 8.4 installed for the **CLI**, but the website/domain was still using PHP 8.3 through **Nginx PHP-FPM**.

CLI check showed:

```bash
php -v
php -r "echo PHP_VERSION.PHP_EOL;"
composer check-platform-reqs
```

Output:

```text
PHP 8.4.21 (cli)
8.4.21
php 8.4.21 success
```

But checking the website in browser:

```text
https://domain.com/php-version.php
```


showed:

```text
8.3.30
```

That means the terminal was using PHP 8.4, but the live website was still running PHP 8.3.

For `php-version.php` see [Step 5: Check Website PHP Version](#step-5-check-website-php-version)

---

## Why This Happens

There are two PHP versions involved:

| Area | PHP Used By |
|---|---|
| Terminal / Composer / Artisan | PHP CLI |
| Website / Browser requests | PHP-FPM through Nginx |

So this command:

```bash
php -v
```

only confirms the terminal PHP version.

It does **not** confirm which PHP version the website is using.

---

## Step 1: Check CLI PHP Version

Run:

```bash
php -v
php -r "echo PHP_VERSION.PHP_EOL;"
```

Expected result:

```text
8.4.x
```

Example:

```text
PHP 8.4.21 (cli)
8.4.21
```

---

## Step 2: Check Composer Platform Requirements

Run inside your project:

```bash
cd /pathone/domain
composer check-platform-reqs
```

If PHP is correct for CLI, you should see something like:

```text
php 8.4.21 success
```

If Composer asks:

```text
Do not run Composer as root/super user!
Continue as root/super user [yes]?
```

It means you are running Composer as `root`.

For production, it is better to run Composer as the project/server user, but for quick debugging you can continue carefully.

---

## Step 3: Fix Git Dubious Ownership Error

If Git shows:

```text
fatal: detected dubious ownership in repository at '/pathone/domain'
```

Run:

```bash
git config --global --add safe.directory /pathone/domain
```

Then go back to your project:

```bash
cd /pathone/domain
```

---

## Step 4: Ignore This Composer Message

If this command:

```bash
composer config platform.php
```

shows:

```text
platform.php is not defined.
```

That is not the main issue.

It only means `composer.json` does not have a fixed platform PHP version like this:

```json
"config": {
  "platform": {
    "php": "8.4.21"
  }
}
```

Usually, you do **not** need to add this on the server.

The important thing is that the actual CLI and web PHP versions are correct.

---

## Step 5: Check Website PHP Version

Create a temporary PHP version file:

```bash
echo "<?php echo PHP_VERSION;" > public/php-version.php
```

Open this in browser:

```text
https://domain.com/php-version.php
```

If it shows:

```text
8.3.30
```

then the website is still running PHP 8.3.

After testing, remove the file:

```bash
rm public/php-version.php
```

---

## Step 6: Check Nginx PHP-FPM Socket

Run:

```bash
grep -R "fastcgi_pass" /etc/nginx/sites-enabled/
```

Problem example:

```text
/etc/nginx/sites-enabled/my-website.com: fastcgi_pass unix:/run/php/php8.3-fpm.sock;
```

This means Nginx is still sending website requests to PHP 8.3.

---

## Step 7: Update Nginx Site Config to PHP 8.4

Open your Nginx site file:

```bash
sudo nano /etc/nginx/sites-enabled/my-website.com
```

Find this line:

```nginx
fastcgi_pass unix:/run/php/php8.3-fpm.sock;
```

Change it to:

```nginx
fastcgi_pass unix:/run/php/php8.4-fpm.sock;
```

Save the file.

---

## Step 8: Test and Restart Services

Test Nginx config:

```bash
sudo nginx -t
```

If it shows success, restart services:

```bash
sudo systemctl restart php8.4-fpm
sudo systemctl restart nginx
```

Optional: if PHP 8.3 is no longer needed for any other site, you can stop it:

```bash
sudo systemctl stop php8.3-fpm
```

Only do this if no other domain depends on PHP 8.3.

---

## Step 9: Confirm Website PHP Version Again

Create the temporary test file again:

```bash
echo "<?php echo PHP_VERSION;" > public/php-version.php
```

Open:

```text
https://domain.com/php-version.php
```

Expected output:

```text
8.4.21
```

Then remove the test file:

```bash
rm public/php-version.php
```

---

## Step 10: Clear Laravel Cache and Reinstall Composer Dependencies

Run:

```bash
cd /pathone/domain
php artisan optimize:clear
composer install --no-dev --optimize-autoloader
```

If needed, also run:

```bash
composer dump-autoload
```

---

## Final Checklist

Run these checks:

```bash
php -v
php -r "echo PHP_VERSION.PHP_EOL;"
composer check-platform-reqs
grep -R "fastcgi_pass" /etc/nginx/sites-enabled/
```

Expected:

```text
CLI PHP: 8.4.x
Composer PHP requirement: success
Nginx socket: php8.4-fpm.sock
Browser PHP version: 8.4.x
```

---

## Summary

The issue was not `composer.json`.

The issue was:

```nginx
fastcgi_pass unix:/run/php/php8.3-fpm.sock;
```

Nginx was still using PHP 8.3 for the website.

After changing it to:

```nginx
fastcgi_pass unix:/run/php/php8.4-fpm.sock;
```

and restarting:

```bash
sudo systemctl restart php8.4-fpm
sudo systemctl restart nginx
```

the domain started using PHP 8.4, and the Composer platform error was resolved.

---

## Important Note

Never leave this file on production:

```text
public/php-version.php
```

Remove it after checking:

```bash
rm public/php-version.php
```
