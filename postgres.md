---
author:
  - Mo D Jabeen
date: 2023-10-01
title: Postgresql
---

# Start postgresql server

Postgres users roles which handle authentication and authorization.

There is a server/client relationship model. Choosing the database name and port to host the a new database. Begin a service to run a database via brew.

-    slash l list databases
-    slash d list tables in database
-   psql -p \[port\] -U \[user\] : this connects to a db
-    i run sql code
-    copy to create CSV file
# Create tables

```sql
CREATE TABLE table_name (
column_name1 col_type (field_length) column_constraints,
column_name2 col_type (field_length),
column_name3 col_type (field_length)
);

CREATE TABLE playground ( #Table name playground
equip_id serial PRIMARY KEY, 
#columnName colType, Primary Key shows this value must be unique
and not null

type varchar (50) NOT NULL, 
#Name type (field length) constraints: not NULL

color varchar (25) NOT NULL,

location varchar(25) check (location in ('north', 'south', 
'west', 'east', 'northeast', 'southeast',
 'southwest', 'northwest')),
#checks value is one of 8 values

install_date date
);

#The length can be implied by the data type
```

## Elements

```sql
INSERT INTO [db_name] (columns) VALUES (values per col);

SELECT * FROM table; Show all values  
```

## Commands

```sql
DROP [database name] # Delete db

SELECT * FROM [] WHERE [] LIKE "" # Return entries like ""

GROUP BY #Group by column value
ORDER BY #Order return by chosen expression
HAVING COUNT #Return with count of chosen expression

NOT NULL #Ensure the entry is not empty
```

Can also use other math aggregate functions.

# Relational Database

Tables can be linked to each other via reference ids.

-   Primary Key : Distinct id value for every entry
-   UNIQUE : Ensures inputs between entries are unique
-   DISTINCT : Query to get distinct entries

```sql
FOREIGN KEY(fk_columns) 
REFERENCES parent_table(parent_key_columns)
#Use the referenced table id to link to the Primary key of parent table
```

-   If a entry is referenced in another table it cannot be deleted
-   Cannot add an entry in parent table if referenced table does not exist
## Join

```sql
JOIN [table] ON table.col = col
#Show referenced entries on two tables

LEFT JOIN #Show full parent table and referenced entries
```

# Data Types

-   bigserial: Autoincrementing eight byte integer
-   inet: host address
-   box: rectangular box on a plane
-   money: currency amount
-   json
-   time: with and without time zone
-   timestamp
# Access via progam

Libpq is the C software used to access postgres via a program.

# Partitioning

Logically separate one large table into smaller physical ones, the main benefit is to increase query speed if one part of the table is heavily accessed.

This done by doing a sequential scan of the relevant partition, instead of a random search through the whole table. The partitioned table itself is a "virtual" table having no storage of its own. Instead, the storage belongs to partitions, which are otherwise-ordinary tables associated with the partitioned table. Not the same as archiving data.

## Types

-   Range: Seperated into a given range of a column or set of cols.
-   List: Key values that should appear in each partition
-   Hash: A modulus, with the remainder of the modulus with the partition key giving the partitions.

## Altering

It is not possible to turn a regular table to partitioned or vice versa. However, you can add an existing regular table as a partition or remove a partition turning it into a regular table.
