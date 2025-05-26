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

## CREATE DATABASE ON DOCKER

1. **Pull the latest postgres image**:

```bash
docker pull postgres
```

1. **Create the database**:

```bash
docker run --name pagila \
	-e POSTGRES_USER=javosw \
	-e POSTGRES_PASSWORD=122333 \
	-e POSTGRES_DB=pagila \
	-d postgres
```

1. **Enter to database**:

```bash
docker exec -it pagila psql -U javosw -d pagila
```

```bash
psql (13.1 (Debian 13.1-1.pgdg100+1))
Type "help" for help.

pagila=#
```

2. **Create all schema objetcs** (tables, etc):

Replace `<local-repo>` by your local directory :
```bash
cat <local-repo>/pagila-schema.sql | docker exec -i postgres psql -U javosw -d pagila
```

3. **Insert all data**:

```bash
cat <local-repo>/pagila-data.sql | docker exec -i postgres psql -U javosw -d pagila
```

## CREATE DATABASE ON DOCKER-COMPOSE

1. Run:

```bash
docker compose up
```

2. Done! Just use:

```bash
docker exec -it pagila psql -U javosw -d pagila
```

```bash
psql (13.1 (Debian 13.1-1.pgdg100+1))
Type "help" for help.

pagila=#
```
## INSTALL NOTE

The `pagila-data.sql` file and the `pagila-insert-data.sql` both contain the same data, the former using `COPY` commands, the latter using `INSERT` commands, so you only need to install one of them. Both formats are provided for those who have trouble using one version or another, and for instructors who want to point out the longer data loading time with the latter. You can load them via **psql**, **pgAdmin**, etc.

Since `JSONB` data is quite large to store on GitHub, the backup is not a plain SQL file. You can still use **psql/pgAdmin**, etc. to load `pagila-schema-jsonb.sql`, however please use `pg_restore` to load `jsonb` data files:

```bash
pg_restore /usr/share/pagila/pagila-data-yum-jsonb.sql -U postgres -d pagila
pg_restore /usr/share/pagila/pagila-data-apt-jsonb.sql -U postgres -d pagila
```

## pgAdmin
pgAdmin is included in the `docker-compose.yml`.

Navigate to the URL: [`http://localhost:5402/`](http://localhost:5402/)
- Default Username: `admin@admin.com`
- Default Password: `122333`

## EXAMPLE QUERY

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
