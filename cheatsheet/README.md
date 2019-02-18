# psql command line cheat sheet


## The psql command line utility

Many administrative tasks can or should be done on your local machine, 
even though if database lives on the cloud.
You can do some of them through a visual user interface, but that's not covered here. 
Knowing how to perform these operations on the command line means you can script them,
and scripting means you can automate tests, error checking, and so on. 

This section isn't a full cheat sheet for `psql`.
It covers the most common operations and shows them roughly in sequence, 
as you'd use them in a typical work session.

Starting and quitting the psql interactive terminal |
----- |
[Command-line prompts for psql](#psql_prompts) |
[Opening a connection to a remote SQL instance](#open_remote_connection) |
[mysql CLI shortcut commands](appendix-b.md#mysql_cli_shortcuts) |

Creating and using databases |
-------------------------------------- |
[Creating a database (CREATE DATABASE)](#creating-a-database) ) |
[Getting a list of databases (SHOW DATABASES)](#show_databases) |
[Choosing a database to work on (USE)](#use_database) |
[Getting a list of tables contained in this database (SHOW TABLES)](#show_tables) |
[Getting a list of fields in a table (SHOW COLUMNS FROM)](#show_columns_from) |

Creating and using tables and records |
------- |
[Creating a table (CREATE TABLE)](#create_table) |
[Adding a record (INSERT INTO)](#insert_record) |
[Doing a simple query--get a list of records (SELECT)](#select) |
[Inserting several records at once (INSERT INTO)](#insert_multiple_records) |
[Adding only specific fields from a record](#insert_specific_fields) |
[Specifying multiple fields](#specify_multiple_fields) |
[Using backslash to enter special characters](#backslash_special_characters) |

Diagnostic commands |
---- |
[Showing the names of all users](#show_all_users) |
[Getting server information](#server_information) |
[Learning the local port number](#port_number) |
[Learning the hostname of your computer](#show_host_name) |
[Getting the name of the current user (SELECT USER())](#select_user) |

## What you need to know

Before using this section, you'll need:

* The user name and password for your PostgreSQL database
* The [IP address of your remote instance](configuration.md#determining_ip_address)
* Possibly, [your own IP address](configuration.md#determining_your_ip_address) if you have difficulty connecting.


### Command-line prompts on the operating system

The `$` starting a command line in the examples below represents your operating system prompt. 
Prompts are configurable so it may well not look like this. 
On Windows it might look like `C:\Program Files\PostgreSQL>` but Windows prompts are also configurable.

````
$ psql -U sampleuser -h localhost
````

A line starting with `#` represents a comment. Same for everything to the right of a `#`. 
If you accidentally type it or copy and paste it in, don't worry. Nothing will happen.

````
This worked to connect to Postgres on DigitalOcean
# -U is the username (it will appear in the \l command)
# -h is the name of the machine where the server is running.
# -p is the port where the database listens to connections. Default is 5432.
# -d is the name of the database to connect to. I think DO generated this for me, or maybe PostgreSQL.
# Password when asked is csizllepewdypieiib
$ psql -U doadmin -h production-sfo-test1-do-user-4866002-0.db.ondigitalocean.com -p 25060 -d mydb

# Open a database in a remote location.
$ psql -U sampleuser -h production-sfo-test1-do-user-4866002-0.db.ondigitalocean.com -p 21334
````
## Using psql

You'll use `psql` (aka the [PostgreSQL interactive terminal](https://www.postgresql.org/docs/current/app-psql.html) most of all because it's used to create databases and tables, show information about tables, and even to enter information (records) into the database.

### Opening a connection locally

A common case during development is 
````
# Log into Postgres as the user named postgres
$ psql -U postgres
````

### Opening a connection remotely

To connect your remote PostgreSQL instance from your local machine, use `psql` at your operating system command line.
Here's a typical connection.

````
# -U is the username (it will appear in the \l command)
# -h is the name of the machine where the server is running.
# -p is the port where the database listens to connections. Default is 5432.
# -d is the name of the database to connect to. I think DO generated this for me, or maybe PostgreSQL.
$ psql -U doadmin -h production-sfo-test1-do-user-4866002-0.db.ondigitalocean.com -p 25060 -d defaultdb
````

Here you'd enter the password. In case someone is peering over your shoulder, the characters are hidden. After you've entered your information properly you'll get this message (truncated for clarity):

### Looking at the psql prompt
A few things appear, then the `psql` prompt is displayed.
The name of the current database appears before the prmopt.

````
psql (11.1, server 11.0)
Type "help" for help.


postgres=# 
````

At this point you're expected to type commands and parameters into the command line.

### Quitting pqsql

Before we learn anything else, here's how to quit `psql` and return to the operating system prompt.
You type backslash, the letter `q`, and then you press the Enter or return key.

````
# Press enter after typing \q
# Remember this is backslash, not forward slash
postgres=# \q
````

This takes you back out to the operating system prompt.

### psql vs SQL commands

`psql` has two different kinds of commands. Those starting with a backslash
are for `psql` itself, as illustrated by the use of `\q` to quit.

Those starting with valid SQL are of course interactive SQL used to
create and modify PostgreSQL databases.

### Warning: SQL commands end with a semicolon!

One gotcha is that almost all SQL commands you enter into `psql` must end in a semicolon. For example:

````
postgres=# DROP TABLE "sample_property";
````

It's easy to forget. If you do forget the semicolon, you'll see this perplexing prompt:

````
[postgres=# DROP TABLE "sample_property"
postgres=#

````

When you do, just remember to finish it off with that semicolon:

````
[postgres=# DROP TABLE "sample_property"
postgres=# ;
````

## Getting information about databases

### \h Help

````
# Get help. Note it's a backslash, not a forward slash.
postgres=# \h
````
You'll get a long list of commands, then output is paused:

````
Available help:
  ABORT                            CREATE USER
  ...
  ALTER AGGREGATE                  CREATE USER MAPPING
  ALTER PROCEDURE                  DROP INDEX
:
````

* Press space to continue, or q to stop the output.

### \l List databases

What most people think of as a database (say, a list of customers) is actually a table. A database is a group of tables, information about those tables, information about users and their permissions, and much more. Some of these databases (and the tables within) are updated automatically by PostgreSQL as you use them.

To get a list of all databases:

````txt
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 visitor   | tom      | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 markets   | tom      | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 tom       | tom      | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 

````
### \c Connect to a database

To see what's inside a database, connect to it using `\c` followed by the database name. 
The prompt changes to match the name of the database you're connecting to.
(The one named `postgres` is always interesting.) Here we're connecting to the one named
`markets`:

````txt
postgres=# \c markets
psql (11.1, server 11.0)
You are now connected to database "markets" as user "tom".
markets=# 
````

### \dt Display tables

* Use `\dt` to list all the tables (technically, *relations*) in the database:

````txt
markets=# \dt

                    List of relations
 Schema |             Name             | Type  |  Owner   
--------+------------------------------+-------+----------
 public | addresspool                  | table | tom
 public | adlookup                     | table | tom
 public | bidactivitysummary           | table | tom
 public | bidactivitydetail            | table | tom
 public | customerpaymentsummary       | table | tom
 ...

````

* If you choose a database such as `postgres` there could be many tables.
Remember you can pause output by pressing `q`.

### \d and \d+ Display columns (field names) of a table

To view the schema of a table, use `\d` followed by the name of the table. 

* To view the schema of a table named `customerpaymentsummary`, enter

```
markets=# \d customerpaymentsummary

                            Table "public.customerpaymentsummary"
            Column            |            Type             | Collation | Nullable | Default 
------------------------------+-----------------------------+-----------+----------+---------
 usersysid                    | integer                     |           | not null | 
 paymentattemptsall           | integer                     |           |          | 
 paymentattemptsmailin        | integer                     |           |          | 
 paymentattemptspaypal        | integer                     |           |          | 
 paymentattemptscreditcard    | integer                     |           |          | 
 paymentacceptedoutagecredit  | integer                     |           |          | 
 totalmoneyin                 | numeric(12,2)               |           |          | 
 updatewhen1                  | timestamp without time zone |           |          | 
 updatewhen2                  | timestamp without time zone |           |          | 

```

### Creating a database

Before you add tables, you need to create a database to contain those tables.
That's not done with `psql`, but instead it's done with `createdb`
at the operating system command line:

````bash
# Replace markets with your databasse name
$ createdb marketd
# Or if you're feeling brave:
postgres=# CREATE DATABASE TODO;
````

The result should look like this:

````
Query OK, 1 row affected (0.07 sec)
````

## Adding tables and records

### Creating a table (CREATE TABLE)

#### NOTE
Before you create any tables, you must create a [database](#creating-a-database) to contain those tables.

This polite form creates a table only if one by that name is not already present. If you're feeling brave you can omit the `IF NOT EXISTS`

````
postgres=# create table if not exists "product" (id MEDIUMINT NOT NULL AUTO_INCREMENT PRIMARY KEY, `name` VARCHAR(100) NOT NULL DEFAULT '', description TEXT);
````

<a name="insert_record"></a>
### Adding a record (INSERT INTO)

Here's how to add a record, populating every field:

````
# The id field is an automatically assigned
# unique number. The autoincrement means
# that number will be increased by at least
# 1 and assigned to that same field when
# a new record is created.
# Using 0 is a placeholder; MySQL automatically updates the field properly.
postgres=# INSERT INTO product VALUES(0, 'Cool item', 'Does neat stuff');
Query OK, 1 row affected (0.13 sec)

````

<a name="select"></a>
### Doing a simple query--get a list of records (SELECT)

````
postgres=# SELECT * FROM product;
+----+---------------+-----------------+
| id | name | description |
+----+---------------+-----------------+
| 1 | Cool item | Does neat stuff |
+----+---------------+-----------------+

````

For more on SELECT, see the [MySQL SELECT Syntax](https://dev.mysql.com/doc/en/select.html).

<a name="insert_multiple_records"></a>
### Inserting several records at once

````
postgres=# INSERT INTO product VALUES
(0, 'A better item', 'Nicer stuff'),
(0, '3rd item rocks', 'Crazy good'),
(0, '4rth item now', 'Not so hot')
;
````

<a name="insert_specific_fields"></a>
#### Adding only specific fields from a record

You can add records but specify only selected fields (also known as columns). MySQL will use common sense default values for the rest.

In this example, only the `name` field will be populated. The `description` column is left blank, and the `id` column is incremented and inserted.

Two records are added:

```sql
postgres=# INSERT INTO product (name) VALUES
('Replacement sprocket'),
('Power adapter')
;
```

Let's look at our handiwork so far:

````
postgres=# select * from product;
+----+----------------------+-----------------+
| id | name | description |
+----+----------------------+-----------------+
| 1 | Cool item | Does neat stuff |
| 2 | A better item | Nicer stuff |
| 3 | 3rd item rocks | Crazy good |
| 4 | 4rth item now | Not so hot |
| 5 | Replacement sprocket | NULL |
| 6 | Power adapter | NULL |
+----+----------------------+-----------------+
````

<a name="specify_multiple_fields"></a>
### Specifying multiple fields

Here we'll enter selected fields (in this case, `name` and `description`, leaving `id` untouched).

````
postgres=# INSERT INTO product (name, description) VALUES
('Thingamajig', 'Better than last year\'s thingamajig');
````

<a name="backslash_special_characters"></a>
### Using backslash to enter special characters

Note that the description, `Better than last year's thingamajig` , contains an apostrophe (`'`) character which would confuse MySQL. It is escaped using the backslash (`\`) as shown above.

#### Let's see the result of using backslash to escape
````
postgres=# select * from product where id=7;
+----+-------------+---------------------------------------------+
| id | name | description |
+----+-------------+---------------------------------------------+
| 7 | Thingamajig | A real upgrade from last year's thingamajig |
+----+-------------+---------------------------------------------+
1 row in set (0.06 sec)

````

## Diagnostic commands

<a name="show_all_users"></a>
### Showing the names of all users

````
postgres=# select User,Host from mysql.user;
+-----------+-----------+
| User | Host |
+-----------+-----------+
| mysql.sys | localhost |
| root | localhost |
| testuser | localhost |
+-----------+-----------+
3 rows in set (0.00 sec)

````


<a name="server_information"></a>
### Getting server information

This lets you understand things like the name of the currently connected database, whether the connection is SSL, the port on the server side being used to connect to Cloud SQL, etc.

````
postgres=# \s
````

Sample output:

````
mysql Ver 14.14 Distrib 5.6.26, for osx10.8 (x86_64) using EditLine wrapper

Connection id: 8
Current database: sampledatabase
Current user: sampleuser@24.256.250.165
SSL: Not in use
Current pager: stdout
Using outfile: ''
Using delimiter: ;
Server version: 5.6.26 (Google)
Protocol version: 10
Connection: 173.256.255.122 via TCP/IP
Server characterset: utf8
Db characterset: utf8
Client characterset: utf8
Conn. characterset: utf8
TCP port: 3306
Uptime: 1 hour 13 min 52 sec

Threads: 1 Questions: 598 Slow queries: 0 Opens: 34 Flush tables: 1 Open tables: 24 Queries per second avg: 0.134

````

<a name="port_number"></a>
### Learning the local port number

````
postgres=# SHOW VARIABLES WHERE Variable_name = 'port';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| port | 3307 |
+---------------+-------+
1 row in set (0.00 sec)
````

<a name="show_host_name"></a>
### Learning the hostname of your computer

````
postgres=# SHOW VARIABLES WHERE Variable_name = 'hostname';

+---------------+-----------------------------------+
| Variable_name | Value |
+---------------+-----------------------------------+
| hostname | tomsmbairhydra.hsd.wa.comcast.net |
+---------------+-----------------------------------+
1 row in set (0.00 sec)
````

<a name="select_user"></a>
### Getting the name of the current user (SELECT USER())

````
[postgres=> select user();

+---------------+
| user() |
+---------------+
| tom@localhost |
+---------------+
1 row in set (0.00 sec)

````

# Reference

* PostgreSQL offical docs: [Server Administration](https://www.postgresql.org/docs/current/admin.html)
* `psql` (the [PostgreSQL interactive terminal](https://www.postgresql.org/docs/current/app-psql.html) 
<!---
Boilerplate

CREATE TABLE IF NOT EXISTS "rental_property" (id serial primary key);

postgres=# CREATE TABLE IF NOT EXISTS account(
 id serial PRIMARY KEY,
 username VARCHAR (50) UNIQUE NOT NULL,
 email VARCHAR (355) UNIQUE NOT NULL,
 created_on TIMESTAMP NOT NULL,
 last_login TIMESTAMP
);



````
[postgres=> 

````
-->






