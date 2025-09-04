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

Postgresql preferred over mysql if insert amount into db is high. mySQL preferred if mainly used only for read. 
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

CREATE TEMPORARY TABLE #Will be dropped at the end of session
```


# Commands

## Elements

```sql
INSERT INTO [db_name] (columns) VALUES (values per col);

SELECT * FROM table; Show all values  
```


```sql
DROP [database name] # Delete db

SELECT * FROM [] WHERE [] LIKE "" # Return entries like ""

GROUP BY #Group by column value
ORDER BY #Order return by chosen expression
HAVING COUNT #Return with count of chosen expression

NOT NULL #Ensure the entry is not empty
```

Can also use other math aggregate functions:

- SUM
- AVG

## UPSERT

If condition not matched insert else update values.

```sql
INSERT INTO inventory (id, name, price, quantity) 
VALUES (1, 'A', 16.99, 120)
ON CONFLICT(id)
DO UPDATE SET 
	price = EXCLUDED.price, 
	quantity = EXCLUDED.quantity;
```

## SELECT

- SELECT DISTINCT : Return only distinct values, no duplicates
- SELECT COUNT(DISTINCT col): Return number of distinct values in col

### Count

Return the number of rows that match the criteria, needs to be linked to a col.

```sql
SELECT COUNT(col) from TABLE

SELECT COUNT(col) from TABLE where col2=y #Return number of values where chosen col matches value 
```

## AS

Aliases used to give a table or a col or multiple cols a temp name.

## IN

Check if value is equal to list of values.

```sql
SELECT film_id, titleFROM filmWHERE film_id in (1, 2, 3);
```

## Group By

Separates a query into groups, ie gathers all rows into a group by a chosen columns value. This then makes it easy to perform aggregate funcs on each group. 

I.e you want to sum all payments from each customer, group by customer id then sum each of those groups.

```sql

SELECT customer_id, SUM (amount)
FROM payment
GROUP BY customer_id
ORDER BY customer_id;

```
## Having

Filter the groups created by group by. 

```sql
SELECT  
	column1,  
	aggregate_function (column2)
FROM  table_name
GROUP BY  column1
HAVING  condition;
```
# Sub Query 

To perform a sub query used in the main query simply place it in brackets.

Operator below is any operator ie =, != etc

```sql

SELECT select_list 
FROM table1
WHERE columnA operator ( SELECT columnB FROM table2 WHERE condition );

```

If want multiple responses in the subquery use IN 

```sql
SELECT select_list 
FROM table1
WHERE columnA IN ( SELECT columnB FROM table2 WHERE condition );

```

## Correlated sub query

When the sub query references the columns from the outer query and the sub query is evaluated once for each row.

- Use a correlated subquery to perform a query that depends on the values of the current row being processed.

Perform the same function on each row, cost N^2. 

```sql

SELECT select_list
FROM table1 t
WHERE colA operator (
	SELECT colA
	FROM table1
	WHERE colB = t.colB
)
```

## Any

Multiple comparisons from a set of values returned via sub query. Will iterate through all rows and return all vals with ANY match from sub query (think OR op).

```sql
SELECT select_list
FROM table1 t
WHERE colA operator ANY (
	SELECT colA
	FROM table1
)
```


## All

Multiple comparisons from a set of values returned via sub query. Will iterate through all rows and return all vals where all match from sub query (think AND op).

```sql
SELECT select_list
FROM table1 t
WHERE colA operator ALL (
	SELECT colA
	FROM table1
)
```


## Exists

Returns true if row exists, used with subquery to iterate rows and return them if the subquery has a row. 

**NOT EXISTS** does the opposite.

```sql

SELECT select_list
FROM table1 t1
WHERE EXISTS (
	SELECT *
	FROM table2 t2
	WHERE t2.colB = t1.colB
)

```

# Common Table Expression (CTE)

Create temporary named result **table** with a query used in other queries.

```sql
WITH cte_name (cols..) AS (
	query
)

SELECT * FROM cte_name
```

## Recursive Queries

Use the previous result as an input for the next query, until the previous result is empty. Then combine all query results into a table. The inital query used in recursive term is the anchor member result. 

```sql

WITH RECURSIVE cte_name (column1, column2, ...) 
AS ( 
	-- anchor member    
	SELECT select_list FROM table1 WHERE condition    
	
	UNION [ALL] 
	
	-- recursive term    
	SELECT select_list FROM cte_name WHERE recursive_condition
)
SELECT * FROM cte_name;

```

# Operators

### LIKE

Allows queries for specific patterns in a column. Used with wildcards to add variability to the pattern. USE ILIKE for case insensitive pattern.

- % to represent one or more characters
- _ to represent one character

```sql
SELECT * FROM TABLE WHERE COL LIKE "X%" #starts with X in col
SELECT * FROM TABLE WHERE COL ILIKE "X%" #starts with X or x in col
```

### BETWEEN

Query with range of values.

```sql
SELECT * FROM TABLE WHERE X BETWEEN 1 AND 10

```

## Case

If else statement used to set conditions on queries, can be used to create labels and organise queries.

```sql

CASE expression 

WHEN value_1 THEN result_1 
WHEN value_2 THEN result_2 
[WHEN ...]

ELSE else_result

END

```

If no result matching in case and else section will return null.

## Coalesce

List of args, returns first non null.

```sql
COALESCE (argument_1, argument_2, …);
COALESCE(expression,replacement)
```

## ISNULL

Replace null with specified replacement value. 

```sql
ISNULL(expression, replacement)
```

## NULLIF

```sql
NULLIF(argument_1,argument_2);
```

The `NULLIF` function returns a null value if `argument_1` equals to `argument_2`, otherwise, it returns `argument_1`.
# Join

## Inner Join

Combine columns of tables based on a common value between table (generally primary key).

```sql
SELECT (cols from combined table)
FROM table1
INNER JOIN table2 ON colA = colB

```

The above works through each row in table1 and creates a table with combined cols where there is a match.

## Left Join

Give matched results + all other rows in original table filling the second table with nulls!

Same as LEFT OUTER JOIN

```sql
SELECT (cols from combined table)
FROM table1
LEFT JOIN table2 ON colA = colB
```

## Right Join

Give matched results + all other rows in second table filling the original table with nulls!

Same as RIGHT OUTER JOIN

```sql
SELECT (cols from combined table)
FROM table1
RIGHT JOIN table2 ON colA = colB
```

## Full Join

Give matched results and all other rows in both tables filling in nulls in missing data.

Same as FULL OUTER JOIN

```sql
SELECT (cols from combined table)
FROM table1
FULL JOIN table2 ON colA = colB
```


# Combine queries

## Union

Combine the result of two selects IF
- The number and order of cols in each select must match
- Data types must be compatible

```sql
SELECT colA
FROM A
UNION
SELECT colA
From B;
```

To show duplicates use **UNION ALL**

## INTERSECT

Combines select and only returns all rows in both queries. Basically a AND op.

```sql
SELECT colA
FROM A
INTERSECT
SELECT colA
From B;
```

## EXCEPT

Combines select and only returns all rows in the first queries but not in the second. Basically a one sided EX OR.

```sql
SELECT colA
FROM A
EXCEPT
SELECT colA
From B;
```

# Explain

Breaks down based on chosen args, given statement. Used to calculate cost of a query.

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


# Data Types

-   bigserial: Autoincrementing eight byte integer
-   inet: host address
-   box: rectangular box on a plane
-   money: currency amount
-   json
-   time: with and without time zone
-   timestamp

# Window Functions

Peform a function on sections of a table "windows" by creating subsets based on a value.

Aggregate functions can be performed on subsets of a table using GROUP BY.

```sql

select
	subset,
	AVG(val)
from
	table1
INNER JOIN subset_table using (subset_val)
GROUP BY
	susbet;

```

Can create windows for a chosen susbet using OVER and PARTITION.

```sql

SELECT 
	group_name, 
	AVG (price) OVER ( PARTITION BY group_name )
FROM products 
INNER JOIN product_groups USING (group_id);

```

PARTITION BY divides the rows into multiple groups to perform aggregate funcs on.

```sql
window_function(arg1, arg2,..) OVER ( 
[PARTITION BY partition_expression] 
[ORDER BY sort_expression [ASC | DESC] [NULLS {FIRST | LAST }])
```

## Ordering Funcs

- ROW_NUMBER() - Assign seq number to each row in a subset
- RANK() - Rank based on value (if multiple are equal still counter in seq order for next)
- DENSE_RANK() - Rank based on value without increasing seq order for multiple
- FIRST_VALUE() - first row in subset
- LAST_VALUE() - last row in subset

## LAG/LEAD

Used to compare prev or next row with current row.

```sql
LAG (expression [,offset] [,default]) over_clause; 
LEAD (expression [,offset] [,default]) over_clause;
```

# Delete Duplicates

# Transactions

A single unit of work, with one or more operations. Uses ACID principles.

If not specified every query will auto wrapped into a transaction.

- BEGIN : Start transaction
- COMMIT: Apply all changes 
- ROLLBACK: Undo before committing. 

```sql

-- start a transaction 
BEGIN; 

-- insert a new row into the accounts table 
INSERT INTO accounts(name,balance) VALUES('Alice',10000); 

-- commit the change (or roll it back later) 
COMMIT;

```

# Triggers

Automatically execute a function in response to an event (insert, update, delete or truncate).
# Views

A named query stored in sever, can be used to directly query data from.
# Indexes

Increases data retrieval speed by providing a fast way to locate rows.

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

# Server

Install postgres using apt get.

```
sudo apt install postgresql postgresql-contrib
```

Setting up a server to connect to:

Update postgres.conf file in /etc/postgresql/\*/main/postgresql.conf to:

```
listen_address = '*'
```

To allow connections from any address.

Switch to user postgres on server:

```
sudo -i -u postgres
psql
```

Setup firewall rules using ufw:

```
sudo ufw allow postgresql/tcp
```

Adjust file /etc/postgresql/\*/main/pg_hba.conf to: 

```
host  all  all 0.0.0.0/0 scram-sha-256
```

Restart service
```bash
sudo systemctl restart postgresql
```