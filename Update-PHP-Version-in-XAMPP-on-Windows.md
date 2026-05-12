# Update PHP Version in XAMPP on Windows (Manual Upgrade)

This guide explains how to manually upgrade PHP in XAMPP on Windows.

> Tested on:
>
> - Windows
> - XAMPP
> - PHP 8.5 (VS17 x64 Thread Safe)

---

# 1. Backup Old PHP Directory

Rename the existing PHP directory:

```txt
C:\xampp\php
```

to something like:

```txt
C:\xampp\php.8.2
```

This acts as a backup in case something breaks.

---

# 2. Download Latest PHP

Download the latest PHP Thread Safe ZIP version from:

[PHP Downloads for Windows](https://www.php.net/downloads.php?os=windows&osvariant=windows-downloads&version=default&utm_source=chatgpt.com)

Downloaded version:

```txt
VS17 x64 Thread Safe
```

> Important:
>
> Download **Thread Safe (TS)** version for Apache/XAMPP.

---

# 3. Create New PHP Directory

Create a new directory:

```txt
C:\xampp\php
```

Extract/move all downloaded PHP files into this directory.

---

# 4. Create php.ini

Inside the new PHP directory, there will be two files:

```txt
php.ini-development
php.ini-production
```

Rename:

```txt
php.ini-development
```

to:

```txt
php.ini
```

---

# 5. Configure Extension Directory

Inside `php.ini`, find:

```ini
;extension_dir = "./"
```

Replace with:

```ini
extension_dir="C:\xampp\php\ext"
```

---

# 6. Enable Required Extensions

Uncomment or add these commonly used extensions:

```ini
extension=bz2
extension=curl
extension=exif
extension=fileinfo
extension=gd
extension=gettext
extension=mbstring
extension=mysqli
extension=openssl
extension=pdo_mysql
extension=pdo_pgsql
extension=pdo_sqlite
extension=pgsql
extension=zip
```

---

# 7. Configure OpenSSL (Optional but Recommended)

If you use:
- Composer
- APIs
- cURL
- HTTPS requests

Find:

```ini
[openssl]
```

Add:

```ini
openssl.cafile="C:\xampp\apache\bin\curl-ca-bundle.crt"
```

Creating certificate like this `curl-ca-bundle.crt` is a different topic.

Also configure cURL:

```ini
[curl]
curl.cainfo="C:\xampp\apache\bin\curl-ca-bundle.crt"
```

---

# 8. Configure PHP Error Logs (Recommended)

Inside `php.ini`:

```ini
log_errors = On
error_log="C:\xampp\php\logs\php_error_log"
```

Create directory if it does not exist:

```txt
C:\xampp\php\logs
```

---

# 9. Verify OpenSSL is Enabled

Run:

```bash
php -m | findstr openssl
```

Expected output:

```txt
openssl
```

---

# 10. Verify Loaded php.ini

Run:

```bash
php --ini
```

Verify:

```txt
Loaded Configuration File: C:\xampp\php\php.ini
```

---

# 11. Restart Apache

Completely restart:
- XAMPP
- Apache

---

# 12. Verify PHP Version

CLI:

```bash
php -v
```

Browser:

Create:

```php
<?php phpinfo();
```

Both should show the new PHP version.

---

# 13. Apache Logs

If Apache or PHP does not start, check:

```txt
C:\xampp\apache\logs\error.log
```

This is the most important log file for startup issues.

---

# Common OpenSSL Error Fix

If you see:

```txt
PHP Startup: Unable to load dynamic library 'php_openssl.dll'
```

Make sure:

```ini
extension=openssl
```

NOT:

```ini
extension=php_openssl.dll
```

---

# Useful Commands

Check loaded extensions:

```bash
php -m
```

Check php.ini:

```bash
php --ini
```

Check PHP version:

```bash
php -v
```

Update Composer:

```bash
composer self-update
```

---

# Recommended Extensions for Laravel

```ini
extension=bz2
extension=curl
extension=fileinfo
extension=gd
extension=mbstring
extension=mysqli
extension=openssl
extension=pdo_mysql
extension=zip
```

---

# Additional Recommendation

Install Microsoft Visual C++ Redistributable (2015-2022 x64):

[Microsoft VC++ Redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?utm_source=chatgpt.com)

Required for newer PHP versions like:
- VS16
- VS17

---

# Final Result

Successfully upgraded:
- PHP 8.2 → PHP 8.5
- Composer working
- OpenSSL working
- XAMPP Apache working
- Laravel projects working