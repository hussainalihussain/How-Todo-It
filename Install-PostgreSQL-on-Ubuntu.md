# Why PostgreSQL

We use PostgreSQL because it provides strong support for modern AI-related features, especially vector search through extensions like `pgvector`. In today’s AI-driven applications, storing embeddings and performing similarity search is very important. PostgreSQL makes this easier while still giving us the reliability, flexibility, and powerful querying features of a traditional relational database.


# Install PostgreSQL on Ubuntu

This guide explains how to install PostgreSQL on Ubuntu step by step.

PostgreSQL is a powerful open-source relational database system.  
It is commonly used with Laravel, Django, Node.js, and many backend applications.

> Tested on:
>
> - Ubuntu
> - PostgreSQL
> - Terminal
> - Laravel Project

---

# 1. Update Ubuntu Packages

First update the package list:

```bash
sudo apt update
```

This command refreshes Ubuntu package information.

It makes sure Ubuntu knows about the latest available packages before installing PostgreSQL.

---

# 2. Install PostgreSQL

Install PostgreSQL and extra useful packages:

```bash
sudo apt install postgresql postgresql-contrib -y
```

Explanation:

- `postgresql` installs the PostgreSQL database server
- `postgresql-contrib` installs extra PostgreSQL tools and extensions
- `-y` automatically confirms installation

---

# 3. Check PostgreSQL Status

After installation, check if PostgreSQL is running:

```bash
sudo systemctl status postgresql
```

If it is running, you will see something like:

```txt
active (exited)
```

or:

```txt
active (running)
```

If PostgreSQL is not running, start it manually:

```bash
sudo systemctl start postgresql
```

Enable PostgreSQL to start automatically after server restart:

```bash
sudo systemctl enable postgresql
```

`enable` means PostgreSQL will automatically start again after server reboot.

---

# 4. Check PostgreSQL Version

Run:

```bash
psql --version
```

Example output:

```txt
psql (PostgreSQL) 16.x
```

The exact version depends on your Ubuntu version.

---

# 5. Login as PostgreSQL Default User

PostgreSQL creates a default Linux user named:

```txt
postgres
```

Switch to this user:

```bash
sudo -i -u postgres
```

Now open the PostgreSQL shell:

```bash
psql
```

You should see:

```txt
postgres=#
```

This means you are inside PostgreSQL.

---

# 6. Create a New PostgreSQL User

Inside the PostgreSQL shell, create a new user:

```sql
CREATE USER myuser WITH PASSWORD 'mypassword';
```

Example:

```sql
CREATE USER laravel_user WITH PASSWORD 'secret123';
```

> Important:
>
> Use a strong password on production servers.

---

# 7. Create a New Database

Create a database and assign it to the user:

```sql
CREATE DATABASE mydatabase OWNER myuser;
```

Example:

```sql
CREATE DATABASE laravel_db OWNER laravel_user;
```

This means `laravel_user` will own and manage `laravel_db`.

---

# 8. Give Privileges to User

Run:

```sql
GRANT ALL PRIVILEGES ON DATABASE mydatabase TO myuser;
```

Example:

```sql
GRANT ALL PRIVILEGES ON DATABASE laravel_db TO laravel_user;
```

This gives the user permission to use the database.

---

# 9. Exit PostgreSQL

Exit from PostgreSQL shell:

```sql
\q
```

Exit from the `postgres` Linux user:

```bash
exit
```

---

# 10. Test PostgreSQL Connection

Test connection using your new user and database:

```bash
psql -U myuser -d mydatabase -h localhost
```

Example:

```bash
psql -U laravel_user -d laravel_db -h localhost
```

It will ask for the password.

If login is successful, you will see:

```txt
mydatabase=>
```

or:

```txt
laravel_db=>
```

To exit:

```sql
\q
```

---

# 11. Configure Laravel `.env`

For Laravel, update your `.env` file:

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=secret123
```

Then run migrations:

```bash
php artisan migrate
```

---

# 12. Install PostgreSQL PHP Extension for Laravel

If Laravel shows a PostgreSQL driver error, install the PHP PostgreSQL extension:

```bash
sudo apt install php-pgsql -y
```

Then restart your web server.

For Apache:

```bash
sudo systemctl restart apache2
```

For Nginx with PHP-FPM, example:

```bash
sudo systemctl restart php8.3-fpm
sudo systemctl restart nginx
```

> Note:
>
> Replace `php8.3-fpm` with your installed PHP-FPM version if different.

---

# 13. Common PostgreSQL Commands

Login as default PostgreSQL user:

```bash
sudo -i -u postgres
```

Open PostgreSQL shell:

```bash
psql
```

List databases:

```sql
\l
```

List users:

```sql
\du
```

Connect to a database:

```sql
\c database_name
```

Exit PostgreSQL shell:

```sql
\q
```

Check PostgreSQL service:

```bash
sudo systemctl status postgresql
```

Restart PostgreSQL:

```bash
sudo systemctl restart postgresql
```

---

# 14. Common Errors

## Error: psql command not found

This means PostgreSQL client is not installed correctly.

Run:

```bash
sudo apt install postgresql-client -y
```

---

## Error: Peer authentication failed

This usually happens when trying to login without using password authentication correctly.

Use:

```bash
psql -U myuser -d mydatabase -h localhost
```

The `-h localhost` forces password-based login.

---

## Error: Laravel could not find PostgreSQL driver

Install PHP PostgreSQL extension:

```bash
sudo apt install php-pgsql -y
```

Then restart your server.

---

# Final Result

PostgreSQL is now installed on Ubuntu.

You have:

- PostgreSQL server installed
- PostgreSQL service running
- New database user created
- New database created
- Laravel `.env` configured
- Useful PostgreSQL commands available

---

# Quick Command Summary

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
sudo systemctl status postgresql
psql --version
sudo -i -u postgres
psql
```

Inside PostgreSQL:

```sql
CREATE USER laravel_user WITH PASSWORD 'secret123';
CREATE DATABASE laravel_db OWNER laravel_user;
GRANT ALL PRIVILEGES ON DATABASE laravel_db TO laravel_user;
\q
```

Back in terminal:

```bash
exit
psql -U laravel_user -d laravel_db -h localhost
```

Laravel `.env`:

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=secret123
```
