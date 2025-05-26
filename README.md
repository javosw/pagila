# Pagila

> Fork is based on **Pagila version 3.1.0**.

> Pagila has been tested against **PostgreSQL 12 and above**.

Pagila started as a port of the [Sakila](https://dev.mysql.com/doc/sakila/en/) example database available for MySQL, which was originally developed by Mike Hillyer of the MySQL AB documentation team. It is intended to provide a standard schema that can be used for examples in books, tutorials, articles, samples, etc.

All the tables, data, views, and functions have been ported; some of the changes made were:

- Changed `char(1)` true/false fields to proper `boolean` fields
- The `last_update` columns were set with triggers to update them
- Added foreign keys
- Removed `DEFAULT 0` on foreign keys since it's pointless with real FK's
- Used PostgreSQL built-in fulltext searching for fulltext index. Removed the need for the `film_text` table.
- The `rewards_report` function was ported to a simple SRF
- Added `JSONB` data

**Fulltext functionality** is built in PostgreSQL, so parts of the schema exist in the main schema file. Example usage:

```sql
SELECT * FROM film WHERE fulltext @@ to_tsquery('fate&india');
```

The `payment` table is designed as a **partitioned table** with a 7 month timespan for the date ranges.

## Create database on Docker

1. **Pull the latest postgres image**:

```bash
docker pull postgres:17.5
```

2. **Create the database**:

```bash
docker run --name pagila \
	-e POSTGRES_USER=javosw \
	-e POSTGRES_PASSWORD=122333 \
	-e POSTGRES_DB=pagila \
	-d postgres
```

3. **Enter to database**:

```bash
docker exec -it pagila psql -U javosw -d pagila
```

```
psql (17.5 (Debian 17.5-1.pgdg120+1))
Type "help" for help.

pagila=#
```

4. **Create all schema objects** (tables, etc):

```bash
docker exec -i pagila psql -U javosw -d pagila < ./sql/main/pagila-schema.sql
```

5. **Insert all data**:

```bash
docker exec -i pagila psql -U javosw -d pagila < ./sql/main/pagila-data.sql
```

## Create database on Docker Compose

1. Run:

```bash
docker compose up
```

2. Done! Just use:

```bash
docker exec -it pagila psql -U javosw -d pagila
```

```
psql (17.5 (Debian 17.5-1.pgdg120+1))
Type "help" for help.

pagila=#
```
## Installation notes

### Loading Standard Data Files
The `pagila-data.sql` file and the `pagila-insert-data.sql` both contain the same data, the former using `COPY` commands (faster for bulk loading), the latter using `INSERT` commands (slower), so you only need to install one of them. Both formats are provided for those who have trouble using one version or another, and for instructors who want to point out the longer data loading time with the latter. You can load them via `psql`.

### Loading JSONB Files
* Schema (plain SQL):
  * `sql/dump/pagila-schema-jsonb.sql`

* Data (binary dump, not plain SQL):
  * `sql/dump/pagila-data-yum-jsonb.sql`
  * `sql/dump/pagila-data-apt-jsonb.sql`
  * `sql/dump/pagila-insert-data_yum-jsonb.sql`
  * `sql/dump/pagila-insert-data_apt-jsonb.sql`


JSONB dumps are large and complex. To reduce GitHub repo size and retain full structure, they were exported in **custom format** using `pg_dump --format=custom`.

Use `pg_restore` when the file is a **binary dump**. You can still use **`psql`** to load **`pagila-schema-jsonb.sql`**, **however** please **use `pg_restore` to load `jsonb` data files**:

```bash
docker exec -i pagila psql -U javosw -d pagila < ./sql/dump/pagila-schema-jsonb.sql
docker exec -i pagila pg_restore --no-owner -U javosw -d pagila < ./sql/dump/pagila-data-apt-jsonb.sql
```

## pgAdmin
pgAdmin is included in the `docker-compose.yml`.

Navigate to the URL: [`http://localhost:5402/`](http://localhost:5402/)
- Default Username: `admin@admin.com`
- Default Password: `122333`

## Sample query

Find late rentals:

```sql
SELECT
	CONCAT(customer.last_name, ', ', customer.first_name) AS customer,
	address.phone,
	film.title
FROM
	rental
	INNER JOIN customer ON rental.customer_id = customer.customer_id
	INNER JOIN address ON customer.address_id = address.address_id
	INNER JOIN inventory ON rental.inventory_id = inventory.inventory_id
	INNER JOIN film ON inventory.film_id = film.film_id
WHERE
	rental.return_date IS NULL
	AND rental_date < CURRENT_DATE
ORDER BY
	title
LIMIT 5;
```
