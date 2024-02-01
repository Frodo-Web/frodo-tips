# PostgreSQL
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
yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum list postgresql*

yum install postgresql12-server.x86_64
```
### Configure PostgreSQL after install
You have to create a new PostgreSQL database cluster before you can use your Postgres database. A database cluster is a collection of databases that are managed by a single server instance. Creating a database cluster consists of creating the directories in which the database data will be placed, generating the shared catalog tables, and creating the template1 and postgres databases.

The template1 database is needed to create a new database. Everything that is stored in it will be placed in a new database when it is created. A postgres database is a default database designed for use by users, utilities, and third-party applications.
```
/usr/pgsql-12/bin/postgresql-12-setup initdb
..
Initializing database ... OK

systemctl start postgresql-12
systemctl enable postgresql-12
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
