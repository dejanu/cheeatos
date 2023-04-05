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
* </ins>[Postgres](postgres.md)</ins>


### General info

* PostgreSQL - relational database RDBMS
* Connecto to DB: `psql -U <user_name> oddba -h <DB_HOSTNAME> -d postgres` 
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

* Increase values of the `max_connections` and `shared_buffers` parameter is to directly edit `postgresql.conf` file:
```bash
max_connections = 150 # (change requires restart)
shared_buffers = 256MB # min 128kB # (change requires restart)
```

* PostgreSQL allocates some amount of memory on a per connection basis, typically around 5 - 10 MB per connection
```bash
# show max connections (some of them are reserved) By default, this value is set to 100.
# max_connections is the upper limit for the number of database connections to all databases
SHOW max_connections;

# check the setting
SELECT current_setting('max_connections');

# pg_stat_activity is a table that stores PostgreSQL connection & activity stats. 
# count of active connections or check all sessions
# connection = relationship between a client and DB server, aka communication channel 
# session = period of time, duration between client connecting and disconnecting to/from DB server aka state of the information exchange
# A connection can have multiple sessions
select count(*) from pg_stat_activity;
select * from pg_stat_activity;

# numbackends is the number of connections (backends connected) to a certain database
SELECT datname, numbackends FROM pg_stat_database;
SELECT * FROM pg_stat_database;
SELECT sum(numbackends) FROM pg_stat_database;
SELECT count(distinct(numbackends)) FROM pg_stat_database;

# check idle connections (open connections that are in the idle state, that also have an open transaction)
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

* Client authentication is controlled by a configuration file, which traditionally is named `pg_hba.conf` and is stored in the database cluster's data directory. (HBA stands for host-based authentication.)
```bash
table pg_hba_file_rules ;
```

* Views-is a virtual table that is created from a `SELECT` statement. Unlike a regular table, a view does not store data on its own, but it provides a logical representation of data from one or more tables.
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