# PostgreSQL
### PSQL
```
psql -h 10.228.228.228 -p 5000 -U sentry sentry
```
If all tables in same schema, you can drop them without dropping the database
```
DROP SCHEMA public CASCADE;
CREATE SCHEMA public;

// Default grants
GRANT ALL ON SCHEMA public TO postgres;
GRANT ALL ON SCHEMA public TO public;
```
Show the size of a table
```
sentry=> SELECT pg_size_pretty(pg_total_relation_size('nodestore_node'));
 pg_size_pretty 
----------------
 2440 GB
```
Clean tables with old data 
```
sentry=> DELETE FROM public.nodestore_node WHERE "timestamp" < NOW() - INTERVAL '15 days';
sentry=> VACUUM FULL public.nodestore_node;
```
### Install PostgreSQL on CentOS 7
Exclude postgresql* packages from Base and Update repos
```
vim /etc/yum.repos.d/CentOS-Base.repo
..
[base]
name=CentOS-$releasever - Base
..
exclude=postgresql*

#released updates 
[updates]
name=CentOS-$releasever - Updates
..
exclude=postgresql*
```
Install official postgresql repo and install postgresql you need
```
// Install dependency
yum install epel-release
yum -y install libzstd-devel

yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum list postgresql*

yum install postgresql15-server.x86_64
```
### Configure PostgreSQL after install
You have to create a new PostgreSQL database cluster before you can use your Postgres database. A database cluster is a collection of databases that are managed by a single server instance. Creating a database cluster consists of creating the directories in which the database data will be placed, generating the shared catalog tables, and creating the template1 and postgres databases.

The template1 database is needed to create a new database. Everything that is stored in it will be placed in a new database when it is created. A postgres database is a default database designed for use by users, utilities, and third-party applications.
```
/usr/pgsql-15/bin/postgresql-15-setup initdb
..
Initializing database ... OK

systemctl start postgresql-15
systemctl enable postgresql-15
```
### Creating a new Role
By default, Postgres uses a concept called roles to handle in authentication and authorization. These are, in some ways, similar to regular Unix-style accounts, but Postgres does not distinguish between users and groups and instead prefers the more flexible term role.

Upon installation, Postgres is set up to use ident authentication, meaning that it associates Postgres roles with a matching Unix/Linux system account. If a role exists within Postgres, a Unix/Linux username with the same name is able to sign in as that role.

The installation procedure created a user account called postgres that is associated with the default Postgres role. In order to use Postgres, you can log in to that account.

Create role:
```
// createuser command is accessible from postgres USER!!!
sudo -u postgres createuser --interactive
..
Enter name of role to add: manticore
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) y
Shall the new role be allowed to create more new roles? (y/n) n
```
Another assumption that the Postgres authentication system makes by default is that for any role used to log in, that role will have a database with the same name which it can access.

This means that, if the user you created in the last section is called manticore, that role will attempt to connect to a database which is also called manticore by default. You can create the appropriate database with the createdb command.

Create db:
```
sudo -u postgres createdb manticore
```
To log in with ident based authentication, you’ll need a Linux user with the same name as your Postgres role and database.
```
adduser manticore
sudo -u manticore psql

manticore=> \conninfo
You are connected to database "manticore" as user "manticore" via socket in "/var/run/postgresql" at port "5432".
```
### Creating table
```sql
CREATE TABLE playground (
    equip_id serial PRIMARY KEY,
    type varchar (50) NOT NULL,
    color varchar (25) NOT NULL,
    location varchar(25) check (location in ('north', 'south', 'west', 'east', 'northeast', 'southeast', 'southwest', 'northwest')),
    install_date date
);

manticore=> \d
                    List of relations
 Schema |          Name           |   Type   |   Owner   
--------+-------------------------+----------+-----------
 public | playground              | table    | manticore
 public | playground_equip_id_seq | sequence | manticore
```
Your playground table is here, but there’s also something called playground_equip_id_seq that is of the type sequence. This is a representation of the serial type that you gave your equip_id column. This keeps track of the next number in the sequence and is created automatically for columns of this type.

If you want to see just the table without the sequence, you can type:
```sql

manticore=> \dt
            List of relations
 Schema |    Name    | Type  |   Owner   
--------+------------+-------+-----------
 public | playground | table | manticore
```
Adding and querying data
```sql
manticore=> INSERT INTO playground (type, color, location, install_date) VALUES ('slide', 'blue', 'south', '2017-04-28');
INSERT 0 1

manticore=> INSERT INTO playground (type, color, location, install_date) VALUES ('swing', 'yellow', 'northwest', '2018-08-16');
INSERT 0 1

manticore=> SELECT * FROM playground;
 equip_id | type  | color  | location  | install_date 
----------+-------+--------+-----------+--------------
        1 | slide | blue   | south     | 2017-04-28
        2 | swing | yellow | northwest | 2018-08-16
```
Another thing to keep in mind is that you do not enter a value for the equip_id column. This is because this is automatically generated whenever a new row in the table is created.
### PostgreSQL dump examples
```
pg_dump -c -Fc -d postgres --exclude-table=nodestore_node > /tmp/sentry1.dmp
pg_restore -c --if-exists -h 10.228.228.228 --port=5000 -U sentry -d sentry -W < /tmp/sentry.dmp
```
### Конспект dump
```sql
\l -list databases
\c - switch database
\dt - list tables
\dt *.* - list tables in all schemas
\dt public.* - list tables in public schema
\dn - list schemas
\copy (select * from db) to '/home/user/file.csv' (format csv, delimiter ';'); - экспорт запроса в csv
\copy table(id,name,value) FROM '/home/user/file.csv' DELIMITER ',' CSV - импорт
\i /path/to/file - выполнить строки из файла
\o /path/to/file - записать выборку в файл
\sf - смотреть user-defined функции

SELECT * FROM pg_roles;
rolname | rolsuper | rolinherit | rolcreaterole | rolcreatedb | rolcatupdate | rolcanlogin | rolreplication | rolconnlimit | rolpassword | rolvaliduntil
 | rolconfig | rolresqueue | oid | rolcreaterextgpfd | rolcreaterexthttp | rolcreatewextgpfd | rolresgroup 
---------+----------+------------+---------------+-------------+--------------+-------------+----------------+--------------+-------------+--------------
-+-----------+-------------+-----+-------------------+-------------------+-------------------+-------------
 gpadmin | t        | t          | t             | t           | t            | t           | t              |           -1 | ********    |              
 |           |        6055 |  10 | t                 | t                 | t                 |        6438

postgres=# \conninfo
You are connected to database "postgres" as user "gpadmin" via socket in "/tmp" at port "5432".

postgres=# SELECT pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

\di+ i_asset_
-[ RECORD 1 ]----------------------
Schema      | public
Name        | i_asset_
Type        | index
Owner       | nikolay
Table       | asset
Size        | 13 MB
Description |

ERROR: permission denied for relation mnp_data_history
GRANT SELECT ON TABLE data_history TO zabbix_script; Дать права на SELECT на опред таблицу опред юзеру


sync_db=# \sf usr.truncate_empty
CREATE OR REPLACE FUNCTION usr.truncate_empty()
 RETURNS integer
 LANGUAGE plpgsql
AS $function$
...
$function$

select * from pg_namespace where nspname='public';
 nspname | nspowner |              nspacl              
---------+----------+----------------------------------
 public  |       10 | {gpadmin=UC/gpadmin,=UC/gpadmin}
(1 row)

List all schemas from current database:
zabbix=# select * from information_schema.schemata;
  catalog_name   |      schema_name       |      schema_owner      | default_character_set_catalog | default_character_set_schema | default_character_set_name | sql_path 
-----------------+------------------------+------------------------+-------------------------------+------------------------------+----------------------------+----------
                         |                            | 
 zabbix_external | usr                    | user                  |                               |                              |                            | 
 zabbix_external | user                  | user                  |                               |                              |                            | 
 zabbix_external | activemq               | activemq                   |                               |                              |                            | 


select * from pg_catalog.pg_stat_activity LIMIT 10; # Таблица с отслеживание статистики по процессам (запросы, транзакции)
select * from pg_catalog.pg_namespace LIMIT 10;  # Also lists schemas

SELECT * FROM pg_stat_activity WHERE state = 'idle in transaction' AND now()-query_start > '10m'::interval;
```
### Roadmap
```
1. User-defined functions, shemas
2. Indexes
3. WAL
4. Replication
5. Vacuum
6. Backup (pgbackrest?)

```
