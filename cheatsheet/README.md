# psql command line cheat sheet

If you've installeld PostgreSQL, this section serves as both a brief tutorial and as a 
quick reference for the absolute least you need to know about `psql`. It shows how to
do the following at the `psql` prompt:

* [Start and quit `psql`](#opening-a-connection-locally)
* [Get help](#h-help)
* [Get information about databases](#getting-information-about-databases)
* [Create databases]((#creating-a-database))
* [Create tables](#creating-a-table-create-table)
* [INSERT, or add records to a table](#adding-a-record-insert-into)
* [SELECT, to do simple queries](#doing-a-simple-query--get-a-list-of-records-select)

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
[Command-line prompts for psql](#using-psql) |
[Quitting psql](##quitting-pqsql) |
[Opening a connection locally](#opening-a-connection-locally) |
[Opening a connection remotely](#opening-a-connection-remotely) |
[Looking at the psql prompt](#looking-at-the-psql-prompt) |

Getting information about databases |
----- |
[\h Help](#h-help) |
[\l List databases](#l-list-databases) |
[\c Connect to a database](#c-connect-to-a-database) |
[\dt Display tables](#dt-display-tables) |
[\d and \d+ Display columns (field names) of a table](#d-and-d-display-columns-field-names-of-a-table) |
[\du Display user roles](#du-display-user-roles)

Creating and using tables and records |
------- |
[Creating a database](#creating-a-database)
[Creating a table (CREATE TABLE)](#creating-a-table-create-table) |
[Adding a record (INSERT INTO)](#adding-a-record-insert-into) |
[Inserting several records at once (INSERT INTO)](#adding-inserting-several-records-at-once) |
[Adding only specific fields from a record](#adding-only-specific-columns-fields-from-a-record) |
[Doing a simple query--get a list of records (SELECT)](#doing-a-simple-query--get-a-list-of-records-select) |


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

### Quitting pqsql

Before we learn anything else, here's how to quit `psql` and return to the operating system prompt.
You type backslash, the letter `q`, and then you press the Enter or return key.

````
# Press enter after typing \q
# Remember this is backslash, not forward slash
postgres=# \q
````

This takes you back out to the operating system prompt.

### Opening a connection locally

A common case during development is opening a connection to a local database (one on your own machine).
Run `psql` with `-U` (for user name) followed by the name of the database, `postgres` in this example:
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

### psql vs SQL commands

`psql` has two different kinds of commands. Those starting with a backslash
are for `psql` itself, as illustrated by the use of `\q` to quit.

Those starting with valid SQL are of course interactive SQL used to
create and modify PostgreSQL databases.

### Warning: SQL commands end with a semicolon!

One gotcha is that almost all SQL commands you enter into `psql` must end in a semicolon. 

* For example,suppose you want to remove a table named `sample_property_5`. You'd enter this command:

````
postgres=# DROP TABLE "sample_property_5";
````

It's easy to forget. If you do forget the semicolon, you'll see this perplexing prompt.
Note that a `[` has been inserted before the username portion of the prompt, and another
prompt appears below it:

````
[postgres=# DROP TABLE "sample_property_5"
postgres=#

````

When you do, just remember to finish it off with that semicolon:

````
[postgres=# DROP TABLE "sample_property_5"
postgres=# ;
````

## Getting information about databases

These aren't SQL commands so just press Enter after them. Remember that:

* When there's more output than fits the screen, it pauses. Press space to continue
* If you want to halt the output, press `q`.

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

You can get help on a particular item by listing it after the `\h` command. 

* For example, to get help on `DROP TABLE`:

````txt
postgres=# \h drop table
````

You'll get help on just tha item:

````txt
Command:     DROP TABLE
Description: remove a table
Syntax:
DROP TABLE [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ]
````

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

You can get info on a single database by following the `\l` prompt with its name.

* For example, to view information about the `template0` database:

````txt
postgres=# \l template0
````

The output would be:

````txt
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
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
Remember you can pause output by pressing space or halt it by pressing `q`.

### \d and \d+ Display columns (field names) of a table

To view the schema of a table, use `\d` followed by the name of the table. 

* To view the schema of a table named `customerpaymentsummary`, enter

```
markets=# \d customerpaymentsummary

                            Table "public.customerpaymentsummary"
            Column            |            Type             | Collation | Nullable | Default 
------------------------------+-----------------------------+-----------+----------+--------
 usersysid                    | integer                     |           | not null |        
 paymentattemptsall           | integer                     |           |          |        
 paymentattemptsmailin        | integer                     |           |          |        
 paymentattemptspaypal        | integer                     |           |          |        
 paymentattemptscreditcard    | integer                     |           |          |        
 paymentacceptedoutagecredit  | integer                     |           |          |        
 totalmoneyin                 | numeric(12,2)               |           |          |        
 updatewhen1                  | timestamp without time zone |           |          |        
 updatewhen2                  | timestamp without time zone |           |          |        

````

To view more detailed information on a table, use `\d+`:
````txt
markets=# \d customerpaymentsummary

                            Table "public.customerpaymentsummary"
            Column            |            Type             | Collation | Nullable | Default | Storage | Stats target |
------------------------------+-----------------------------+-----------+----------+---------+---------+---------------
 usersysid                    | integer                     |           | not null |         | plain   |              |   
 paymentattemptsall           | integer                     |           |          |         | plain   |              |   
 paymentattemptsmailin        | integer                     |           |          |         | plain   |              |   
 paymentattemptspaypal        | integer                     |           |          |         | plain   |              |   
 paymentattemptscreditcard    | integer                     |           |          |         | plain   |              |   
 paymentacceptedoutagecredit  | integer                     |           |          |         | plain   |              |    
 totalmoneyin                 | numeric(12,2)               |           |          |         | main    |              |    
 updatewhen1                  | timestamp without time zone |           |          |         | plain   |              |   
 updatewhen2                  | timestamp without time zone |           |          |         | plain   |              |    

Indexes:
````

### \du Display user roles

* To view users and their roles, use `\du`:

````txt
postgres=# \du
                                    List of roles
 Role name  |                         Attributes                         | Member of 
------------+------------------------------------------------------------+-----------
 smanager   | Superuser                                                  | {}
 postgres   | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 tom        | Superuser, Create role, Create DB                          | {}
````

* To view the role of a specific user, pass it after the `\du` command.

* To view users and their roles, use `\du`. For example, to see the `tom` role:

````txt
postgres=# \du tom
                                    List of roles
 Role name  |                         Attributes                         | Member of 
------------+------------------------------------------------------------+-----------
 tom        | Superuser, Create role, Create DB                          | {}
````

## Creating a database

Before you add tables, you need to create a database to contain those tables.
That's not done with `psql`, but instead it's done with `createdb`
at the operating system command line:

````bash
# Replace markets with your database name
$ createdb marketd
````
On success, there is no visual feedback. Thanks, PostgreSQL.

## Adding tables and records

### Creating a table (CREATE TABLE)

To add a table schema to the database:

````sql
postgres=# create table if not exists product (
  id              SERIAL,
  name            VARCHAR(100) NOT NULL,
  sku             CHAR(8)
);
````

And `psql` responds with:

````sql
CREATE TABLE
````

### Adding a record (INSERT INTO)

* Here's how to add a record, populating every field:

````sql
# The id field is an automatically assigned
# when you use DEFAULT. The serial primary key means
# that number will be increased by at least
# 1 and assigned to that same field when
# a new record is created.
# Using DEFAULT is a placeholder. 
# In its place PostgreSQL automatically generates a unique integer for it.
postgres=# INSERT INTO product VALUES(DEFAULT, 'Apple, Fuji', '4131');
````

PostgreSQL responds with:

````sql
INSERT 0 1
````

* Try it again and you get a simliar response.

````sql
postgres=# INSERT INTO product VALUES(DEFAULT, 'Banana', '4011');
INSERT 0 1
````

### Adding (inserting) several records at once

* You can enter a list of records using this syntax:

````
postgres=# INSERT INTO product VALUES
(DEFAULT, 'Carrots', 4562),
(DEFAULT, 'Durian', 5228)
;
````


#### Adding only specific (columns) fields from a record

You can add records but specify only selected fields (also known as columns). MySQL will use common sense default values for the rest.

In this example, only the `name` field will be populated. The `sku` column is left blank, and the `id` column is incremented and inserted.

Two records are added:

```sql
postgres=# INSERT INTO product (name) VALUES
('Endive'),
('Figs')
;
```

PostgreSQL responds with the number of records inserted:

````txt
INSERT 0 2
````

### Doing a simple query--get a list of records (SELECT)

* Let's list all the records in the `product` table:

````
postgres=# SELECT * FROM product;
````

The response:

````txt
postgres=# select * from product;
 id |    name     |   sku    
----+-------------+----------
  1 | Apple, Fuji | 4131    
  2 | Banana      | 4011    
(2 rows)
````
For more on SELECT, see the PostgreSQL [SELECT statement](https://www.postgresql.org/docs/current/sql-select.html).


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


-->






