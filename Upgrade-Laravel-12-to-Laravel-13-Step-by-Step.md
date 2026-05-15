# Upgrade Laravel 12 to Laravel 13 Step by Step

This guide explains how to manually upgrade an existing Laravel 12 project to Laravel 13.

> Tested on:
>
> - Windows
> - XAMPP
> - Composer
> - Laravel 12 → Laravel 13
> - PHP 8.3 or higher

This guide also includes my real upgrade experience, including manually updating `composer.json`, moving `composer.lock`, and using Composer ignore-platform flags on Windows/XAMPP.

---

# 1. Create a New Git Branch

Before upgrading, create a separate Git branch so your main branch stays safe.

```bash
git checkout -b upgrade/laravel-13
```

This helps you return to your previous branch if something breaks.

---

# 2. Check PHP Version

Laravel 13 requires PHP 8.3 or higher.

Run:

```bash
php -v
```

Expected:

```txt
PHP 8.3 or higher
```

If your PHP version is older, upgrade PHP first.

---

# 3. Check Current Laravel Version

Run:

```bash
php artisan --version
```

Example output:

```txt
Laravel Framework 12.x
```

---

# 4. Update Laravel Dependencies

Laravel official upgrade documentation recommends updating the Laravel framework dependency.

Basic command:

```bash
composer require laravel/framework:^13.0 -W
```

Also update Laravel Tinker:

```bash
composer require laravel/tinker:^3.0 -W
```

The `-W` flag allows Composer to update related dependencies too.

---

# 5. In Windows / XAMPP

In my case, I was using Windows/XAMPP and Composer showed platform requirement issues for extensions like:

```txt
ext-pcntl
ext-posix
```

`Tip`: This mostly come if you are using [Laravel Horizon](https://github.com/laravel/horizon).

These extensions are usually not available on Windows, so I used this command locally:

```bash
composer require laravel/framework:^13.0 -W --ignore-platform-req=ext-pcntl --ignore-platform-req=ext-posix
```

Also you might use `--ignore-platform-reqs` flag.

> Note:
>
> This is useful for local Windows/XAMPP development.
> Do not blindly use ignore-platform flags on production servers.

---

# 6. Manually Update composer.json

I also manually updated the required package versions inside:

```txt
composer.json
```

Update Laravel framework:

```json
"laravel/framework": "^13.0"
```

Update Laravel Tinker:

```json
"laravel/tinker": "^3.0"
```

Update PHPUnit:

```json
"phpunit/phpunit": "^12.0"
```

If you are using Pest, update Pest too:

```json
"pestphp/pest": "^4.0"
```

---

# 7. Backup or Move composer.lock

For extra safety, I moved the existing `composer.lock` file to another location.

You can rename:

```txt
composer.lock
```

to:

```txt
composer.lock.backup
```

Or move it outside the project temporarily.

> Important:
>
> In team or production projects, do not permanently delete `composer.lock` without understanding the impact.
> After a successful upgrade, the new `composer.lock` should usually be committed.

---

# 8. Install Dependencies Again

After updating `composer.json` and moving/removing `composer.lock`, run:

```bash
composer install
```

This will install dependencies and generate a new `composer.lock` file.

If you want Composer to update all related packages properly, you can also use:

```bash
composer update -W
```

`-W` means `--with-all-dependencies`

For Windows/XAMPP, if platform extension errors appear, use:

```bash
composer update -W --ignore-platform-req=ext-pcntl --ignore-platform-req=ext-posix
```

---

# 9. Clear Laravel Cache

After Composer finishes, clear Laravel cache:

```bash
php artisan optimize:clear
```

You can also run these commands separately:

```bash
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear
```

---

# 10. Run Database Migrations

Run migrations:

```bash
php artisan migrate
```

If your project uses seeders for permissions, prompts, roles, or default settings, run the required seeders:

```bash
php artisan db:seed
```

Or run a specific seeder:

```bash
php artisan db:seed --class=SeederClassName
```

---

# 11. Verify Laravel Version

Run:

```bash
php artisan --version
```

Expected output:

```txt
Laravel Framework 13.x
```

---

# 12. Run Tests

If your project uses PHPUnit:

```bash
php artisan test
```

Or:

```bash
vendor/bin/phpunit
```

If your project uses Pest:

```bash
vendor/bin/pest
```

---

# 13. Start the Project

Run:

```bash
php artisan serve
```

Or if you are using XAMPP, restart Apache and open your project in the browser.

---

# 14. Common Composer Debug Commands

If Laravel 13 cannot be installed, check which package is blocking it:

```bash
composer why-not laravel/framework 13.0
```

Check installed package details:

```bash
composer show laravel/framework
```

Check Laravel project information:

```bash
php artisan about
```

---

# 15. Commit the Upgrade

After everything works, check changed files:

```bash
git status
```

Usually these files will be changed:

```txt
composer.json
composer.lock
```

Commit them:

```bash
git add composer.json composer.lock
git commit -m "Upgrade Laravel 12 to Laravel 13"
```

---

# Final Result

Successfully upgraded:

- Laravel 12 → Laravel 13
- `composer.json` updated manually
- `composer.lock` moved/backed up and regenerated
- Composer dependencies updated
- PHPUnit / Pest updated if required
- Laravel cache cleared
- Project running successfully on Windows/XAMPP

---

# Useful Commands Summary

Create upgrade branch:

```bash
git checkout -b upgrade/laravel-13
```

Update Laravel:

```bash
composer require laravel/framework:^13.0 -W
```

Windows/XAMPP command I used:

```bash
composer require laravel/framework:^13.0 -W --ignore-platform-req=ext-pcntl --ignore-platform-req=ext-posix
```

Install after moving `composer.lock`:

```bash
composer install
```

Clear Laravel cache:

```bash
php artisan optimize:clear
```

Check Laravel version:

```bash
php artisan --version
```
