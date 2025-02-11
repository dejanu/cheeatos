## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](gcp.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* [Kubernetes](k8s.md)
* [Istio](istio.md)
* [OIDC](openID.md)
* <ins>[PostgreSQL](postgres.md)</ins>
* [Terraform](terraform.md)

---

### General info

* PostgreSQL - relational database RDBMS
* Connecto to DB: `psql -U <user_name> oddba -h <DB_HOSTNAME> -d postgres`
* A CONNECTION CAN HAVE MULTIPLE SESSIONS:
```
CONNECTION = relationship between a client and DB server, aka communication channel 
SESSION = period of time, duration between client connecting and disconnecting to/from DB server aka state of the information exchange
```
* Usefull cmds:

```bash
# check db name - check the current db you are connected to
current_database();
SELECT current_database();

# return the current db and user 
\conninfo

# create a new connection \connect
\c

# show databases (+ verbose) To list all the databases created within PostgreSQL Server.
\l or \l+

# connect to a database from a PostgreS 
\connect database_name
\c database_name

# creating a connection from app
const conString = "postgres://YourUserName:YourPassword@YourHostname:5432/YourDatabaseName";

# list the tables in the current database
\dt

# count rows
select count(*) from TABLE_NAME;

# check headers columns for table
\d+ "TABLE_NAME"

# determine the size of a database using pg_database_size() function
SELECT pg_size_pretty( pg_database_size('DB_NAME') );

# take a full dump of table call Test
\copy  (SELECT * FROM "Test") to '/absolute/path/fulldump.csv' (format csv, delimiter';')
```

### Connections

* Increase values of the `max_connections` and `shared_buffers` parameter edit directly `postgresql.conf` file:

```bash
# check default parameters usually are stored in postgresql.conf
SELECT current_setting('effective_cache_size');
SELECT current_setting('shared_buffers');
SELECT current_setting('max_connections');
show effective_cache_size;
show shared_buffers;
show max_connections;
show work_mem;
show maintenance_work_mem;

# changes of the default settings of these configuration params requires a restart

# concurrent connections and limit the scalability of the system
max_connections(100) - maximum number of concurrent connections that the database server can accept from clients

# is critical to PostgreSQL's performance, as it directly affects the amount of data that can be held in memory and reduces the need for disk I/O operations
# As per best practice is shared_buffers = 25% of the memory in the VM/server
shared_buffers(128MB) - the amount of memory allocated for caching data and indexes in shared memory


# helps the query planner to make better decisions about which indexes to use and how to optimize queries
# As per best practice is effective_cache  = RAM * 0.7
effective_cache_size(4GB) # amount of memory the database system expects to be available for disk caching by the operating system and other processes.
```

* PostgreSQL allocates some amount of memory on a per connection basis, typically around 5 - 10 MB per connection:

```bash
# show max connections (some of them are reserved) By default, this value is set to 100.
# max_connections is the upper limit for the number of database connections to all databases
SHOW max_connections;

# check default settings
SELECT current_setting('max_connections');

# pg_stat_activity is a table that stores PostgreSQL connection & activity stats. 

# count of active connections or check all sessions
SELECT datname, usename, COUNT(*) FROM pg_stat_activity where datname not in ('azuresu','postgres') and usename not in ('phiadmin') GROUP BY datname, usename ORDER BY COUNT(datname) DESC;
select count(*) from pg_stat_activity;
select * from pg_stat_activity;

# numbackends is the number of connections (backends connected) to a certain database
SELECT datname, numbackends FROM pg_stat_database;
SELECT * FROM pg_stat_database;
SELECT sum(numbackends) FROM pg_stat_database;
SELECT count(distinct(numbackends)) FROM pg_stat_database;

# check idle connections (open connections that are in the idle state, that also have an open transaction)
select * from pg_stat_activity where (state = 'idle')
select * from pg_stat_activity where (state = 'idle in transaction') and xact_start is not null;
SELECT name, value FROM v$parameter WHERE name = 'sessions';

# kill session using pg_stat_activity
# datname = Name of the database this backend is connected to
select pid from pg_stat_activity where datname = 'vault'; 
SELECT pg_terminate_backend(3278127);

# increase values of the max_connections and shared_buffers parameter is to directly edit postgresql.conf file
max_connections = 150 # (change requires restart)
shared_buffers = 256MB # min 128kB # (change requires restart)
```

* Client authentication is controlled by a configuration file, which traditionally is named `pg_hba.conf` and is stored in the database cluster's data directory. (HBA stands for host-based authentication).

```bash
table pg_hba_file_rules ;
```

* Views - is a virtual table that is created from a `SELECT` statement. Unlike a regular table, a view does not store data on its own, but it provides a logical representation of data from one or more tables.

```bash
# create VIEW
CREATE VIEW employee_salaries AS
SELECT name, salary
FROM employees;

# list all the views in the current schema e.g pg_stat_statements VIEW
\dv

# list views in all schemas
\dvS
```

### Reclaim storage

* Periodic maintenance known as [routine vacuuming](https://www.postgresql.org/docs/12/routine-vacuuming.html)
* Occupied by dead tuples (row/record in a table that has been marked for deletion but has not yet been physically removed from the table) using [vacuum](https://www.postgresql.org/docs/current/sql-vacuum.html)

```bash
# check if autovacuum is enabled
# setting for the autovaccum can be found in /var/lib/postgresql/data/postgresql.conf
SHOW autovacuum;

# check 
SELECT * FROM pg_settings WHERE name LIKE 'autovacuum%';
```
* VACUUM reclaims disk space, but it locks the table for the duration of the operation, making it unavailable for writes:

```bash

# particularly usefull after a large number of deletions
VACUUM FULL;
```

*  [vaccumlo](https://www.postgresql.org/docs/current/vacuumlo.html) client tool removes orphaned large objects(such as binary data stored in `pg_largeobject`):

```bash
# install vacuumlo
apt-get install postgresql-contrib-9.3

vacuumlo --dry-run -U <user> --host=<FQDN for DB> -p 5432 '<DB_NAME>'
```

### Usefull links:

* [Tune pg settings](https://pgtune.leopard.in.ua/)
* [Error codes](https://www.postgresql.org/docs/current/errcodes-appendix.html)
* [ENV vars](https://www.postgresql.org/docs/14/libpq-envars.html) to avoid hard-coding database connection information

* For a dedicated database server:
```bash
# effective_cache_size parameter to a value between 50% and 75% of the total available memory on the system a good setting 
effective_cache_size = RAM * 0.7

# shared_buffers common recommendation is to set it to 25% to 33% of the total available memory on the system a good setting
shared_buffers_size = RAM * 0.3
```
* Queries for Finding the size of various object in the [DB](https://wiki.postgresql.org/wiki/Disk_Usage)