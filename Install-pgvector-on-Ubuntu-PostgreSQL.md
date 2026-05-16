# Install pgvector on Ubuntu PostgreSQL

This guide explains how to install and enable **pgvector** on Ubuntu for PostgreSQL.

`pgvector` is used for vector similarity search inside PostgreSQL.  
It is commonly used for AI search, semantic search, embeddings, RAG systems, and Laravel/PostgreSQL projects.

> Tested on:
>
> - Ubuntu 24.04
> - PostgreSQL 16.13
> - pgvector 0.8.x
> - Laravel project
> - Database example: `your_database_name_goes_here`

---

# Before You Start: Replace Placeholder Names

This guide uses fake placeholder names so you do not accidentally publish your real database, table, or field names.

Replace these values with your own project values:

| Placeholder                       | Replace With                                        |
| --------------------------------- | --------------------------------------------------- |
| `your_database_name_goes_here`    | Your PostgreSQL database name                       |
| `your_table_name_goes_here`       | Your table name                                     |
| `your_embedding_column_goes_here` | Your vector/embedding column name                   |
| `your_filter_column_goes_here`    | Your filter column name, if your index uses `WHERE` |
| `your_filter_value_goes_here`     | Your filter value, if your index uses `WHERE`       |
| `your_index_name_goes_here`       | Your custom PostgreSQL index name                   |
| `your_vector_dimensions_go_here`  | Your vector dimension, for example `1536` or `3072` |

Example:

```txt
your_database_name_goes_here      -> my_app_database
your_table_name_goes_here         -> product_embeddings
your_embedding_column_goes_here   -> embedding
your_filter_column_goes_here      -> model_id
your_filter_value_goes_here       -> 1
your_vector_dimensions_go_here    -> 1536
```

> Important:
>
> Do not commit real production database names, private table names, passwords, API keys, or internal project-specific values in a public repository.

---

# 1. Check PostgreSQL Version

First check your PostgreSQL version:

```bash
psql --version
```

Example output:

```txt
psql (PostgreSQL) 16.13 (Ubuntu 16.13-0ubuntu0.24.04.1)
```

In this example, PostgreSQL version is **16**, so the pgvector package should also use version **16**.

---

# 2. Add PostgreSQL Official APT Repository

Ubuntu’s default repository may provide an older pgvector version.

For example, Ubuntu 24.04 can provide an older `postgresql-16-pgvector` package, but `halfvec` requires pgvector `0.7.0+`.

Add the official PostgreSQL APT repository:

```bash
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
sudo apt update
```

Explanation:

- `postgresql-common` installs PostgreSQL helper scripts
- `apt.postgresql.org.sh` adds the official PostgreSQL APT repository
- `sudo apt update` refreshes the package list

---

# 3. Install pgvector

For PostgreSQL 16, install:

```bash
sudo apt install postgresql-16-pgvector -y
```

If you are using another PostgreSQL version, change the version number.

Example:

```bash
sudo apt install postgresql-17-pgvector -y
```

or:

```bash
sudo apt install postgresql-18-pgvector -y
```

The package version should match your PostgreSQL server version.

---

# 4. Restart PostgreSQL

After installation, restart PostgreSQL:

```bash
sudo systemctl restart postgresql
```

Check status:

```bash
sudo systemctl status postgresql
```

---

# 5. Enable pgvector in Your Database

Installing the package is not enough.

You also need to enable the extension inside the specific database where you want to use it.

Example database placeholder:

```txt
your_database_name_goes_here
```

Open PostgreSQL for that database:

```bash
sudo -u postgres psql -d your_database_name_goes_here
```

Then enable the extension:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Important:

```txt
Extension name is vector, not pgvector.
```

---

# 6. Check Installed pgvector Version

Inside PostgreSQL, run:

```sql
SELECT extname, extversion
FROM pg_extension
WHERE extname = 'vector';
```

Expected example:

```txt
 extname | extversion
---------+------------
 vector  | 0.8.0
```

If it shows an old version like this:

```txt
 vector | 0.6.0
```

then update the extension inside the database:

```sql
ALTER EXTENSION vector UPDATE;
```

Then check again:

```sql
SELECT extname, extversion
FROM pg_extension
WHERE extname = 'vector';
```

---

# 7. Check if halfvec Exists

If your project uses `halfvec`, check it:

```sql
SELECT typname
FROM pg_type
WHERE typname = 'halfvec';
```

Expected result:

```txt
 typname
---------
 halfvec
```

If this returns `0 rows`, then your current database extension version does not support `halfvec`, or the extension has not been updated inside that database.

Run:

```sql
ALTER EXTENSION vector UPDATE;
```

Then check again:

```sql
SELECT typname
FROM pg_type
WHERE typname = 'halfvec';
```

---

# 8. Quick Test

Create a demo test table:

```sql
CREATE TABLE demo_vector_items (
    id bigserial PRIMARY KEY,
    demo_embedding vector(3)
);
```

Insert demo test data:

```sql
INSERT INTO demo_vector_items (demo_embedding)
VALUES ('[1,2,3]'), ('[4,5,6]');
```

Run similarity search:

```sql
SELECT *
FROM demo_vector_items
ORDER BY demo_embedding <-> '[3,1,2]'
LIMIT 5;
```

The `<->` operator is used for L2 distance search.

Optional cleanup:

```sql
DROP TABLE IF EXISTS demo_vector_items;
```

---

# 9. Example halfvec HNSW Index

If your embedding column is a normal `vector`, but you want to index it as `halfvec`, you can use an expression index.

Replace the placeholder names before running this SQL:

```sql
CREATE INDEX your_index_name_goes_here
ON your_table_name_goes_here
USING hnsw ((your_embedding_column_goes_here::halfvec(your_vector_dimensions_go_here)) halfvec_cosine_ops)
WHERE (your_filter_column_goes_here = your_filter_value_goes_here AND your_embedding_column_goes_here IS NOT NULL);
```

Example using fake/demo names:

```sql
CREATE INDEX demo_items_model_hnsw_idx
ON demo_items
USING hnsw ((demo_embedding::halfvec(3072)) halfvec_cosine_ops)
WHERE (demo_model_id = 1 AND demo_embedding IS NOT NULL);
```

This kind of index is useful when:

- embeddings are large
- you want cosine similarity
- you want smaller index size
- you want to use half-precision vectors

> Note:
>
> If you do not need a filtered/partial index, remove the `WHERE (...)` line.

---

# 10. Laravel Migration Example

In Laravel, first enable the extension:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Support\Facades\DB;

return new class extends Migration
{
    public function up(): void
    {
        DB::statement('CREATE EXTENSION IF NOT EXISTS vector');
    }

    public function down(): void
    {
        DB::statement('DROP EXTENSION IF EXISTS vector');
    }
};
```

For your HNSW index:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Support\Facades\DB;

return new class extends Migration
{
    public function up(): void
    {
        DB::statement('CREATE EXTENSION IF NOT EXISTS vector');

        DB::statement("
            CREATE INDEX your_index_name_goes_here
            ON your_table_name_goes_here
            USING hnsw ((your_embedding_column_goes_here::halfvec(your_vector_dimensions_go_here)) halfvec_cosine_ops)
            WHERE (your_filter_column_goes_here = your_filter_value_goes_here AND your_embedding_column_goes_here IS NOT NULL)
        ");
    }

    public function down(): void
    {
        DB::statement('DROP INDEX IF EXISTS your_index_name_goes_here');
    }
};
```

Fake/demo example:

```php
DB::statement("
    CREATE INDEX demo_items_model_hnsw_idx
    ON demo_items
    USING hnsw ((demo_embedding::halfvec(3072)) halfvec_cosine_ops)
    WHERE (demo_model_id = 1 AND demo_embedding IS NOT NULL)
");
```

---

# 11. Problem Faced

While running a Laravel migration, this error appeared:

```txt
SQLSTATE[42704]: Undefined object: 7 ERROR: type "halfvec" does not exist
```

The failing SQL was similar to this:

```sql
CREATE INDEX your_index_name_goes_here
ON your_table_name_goes_here
USING hnsw ((your_embedding_column_goes_here::halfvec(your_vector_dimensions_go_here)) halfvec_cosine_ops)
WHERE (your_filter_column_goes_here = your_filter_value_goes_here AND your_embedding_column_goes_here IS NOT NULL);
```

---

# 12. Why This Problem Happened

The database had pgvector enabled, but the enabled extension version was old:

```sql
SELECT extname, extversion
FROM pg_extension
WHERE extname = 'vector';
```

Output:

```txt
 vector | 0.6.0
```

Then this check was run:

```sql
SELECT typname
FROM pg_type
WHERE typname = 'halfvec';
```

Output:

```txt
0 rows
```

This confirmed that `halfvec` was not available in the current database.

Reason:

```txt
pgvector 0.6.0 does not include halfvec.
halfvec was added in pgvector 0.7.0.
```

---

# 13. How It Was Solved

The extension was updated inside the actual database:

```sql
ALTER EXTENSION vector UPDATE;
```

After that, this command was run again:

```sql
SELECT typname
FROM pg_type
WHERE typname = 'halfvec';
```

Now it returned:

```txt
halfvec
```

Then pgvector version was checked again and it was updated to:

```txt
0.8.x
```

So the final fix was:

```sql
ALTER EXTENSION vector UPDATE;
```

But this only worked because a newer pgvector package was already installed or available on the server.

---

# 14. Important Lesson

There are two different things:

## Package installed on Ubuntu

This installs pgvector files on the server:

```bash
sudo apt install postgresql-16-pgvector -y
```

## Extension enabled/updated inside database

This enables or updates pgvector inside one specific database:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
ALTER EXTENSION vector UPDATE;
```

If the package is updated but the database extension is still old, PostgreSQL may still show old behavior.

That is why `halfvec` was missing until the extension was updated inside `your_database_name_goes_here`.

---

# 15. Useful Debug Commands

Check PostgreSQL version:

```bash
psql --version
```

Check installed pgvector package:

```bash
apt-cache policy postgresql-16-pgvector
```

Check PostgreSQL service:

```bash
sudo systemctl status postgresql
```

Open database:

```bash
sudo -u postgres psql -d your_database_name_goes_here
```

Check pgvector extension version:

```sql
SELECT extname, extversion
FROM pg_extension
WHERE extname = 'vector';
```

Check if `halfvec` exists:

```sql
SELECT typname
FROM pg_type
WHERE typname = 'halfvec';
```

Update extension:

```sql
ALTER EXTENSION vector UPDATE;
```

List all installed extensions:

```sql
\dx
```

Exit PostgreSQL:

```sql
\q
```

---

# 16. Commands to Check What You Recently Ran

Check recent shell commands:

```bash
history | tail -n 100
```

If history does not show enough:

```bash
cat ~/.bash_history | tail -n 100
```

Check recent apt commands:

```bash
cat /var/log/apt/history.log | tail -n 100
```

Filter PostgreSQL and pgvector commands:

```bash
grep -i "pgvector\|postgresql-16-pgvector\|postgresql\|apt.postgresql" /var/log/apt/history.log
```

Check PostgreSQL shell history:

```bash
sudo -u postgres bash -lc 'tail -n 100 ~/.psql_history'
```

---

# Final Summary

In this setup:

1. PostgreSQL was installed successfully.
2. pgvector was installed for PostgreSQL 16.
3. The database had `vector` extension version `0.6.0`.
4. The migration failed because `halfvec` did not exist.
5. `ALTER EXTENSION vector UPDATE;` updated the database extension.
6. pgvector became version `0.8.x`.
7. `halfvec` became available.
8. The HNSW index using `halfvec_cosine_ops` could now work.

---

# References

- pgvector GitHub: https://github.com/pgvector/pgvector
- pgvector 0.7.0 release: https://www.postgresql.org/about/news/pgvector-070-released-2852/
