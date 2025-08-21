---
author:
  - Mo D Jabeen
date: 2025-08-21
---

Designed for high performance and efficiency on time series data. With very fast ingestion (write) and simplified query methods.

# Basics

Does not allow creation of new databases, all tables are created in the default qdp database. In QuestDB, you operate within a single namespace.

The web console allows easy visualisation and csv upload.

Web console port: 9000

PSQL 

Can use standard psql clients.

*QuestDB is **not** a PostgreSQL database but is **compatible with the [PostgreSQL wire protocol](https://questdb.com/docs/reference/api/postgres/)**. This means you can connect using PostgreSQL-compatible libraries and clients and execute SQL commands. However, compatibility with PostgreSQL system catalogs, metadata queries, data types, and functions is limited.*

```
"host":"localhost",
"port":"8812",
"user":"admin",
"pwd":"quest",
"dbname":"qdb"
```

# Schema

Should choose:
- A designated timestamp to allow for time series data optimisations. 
- A partition resolution to optimise queries at that resolution 

```sql

CREATE TABLE trades (  
timestamp TIMESTAMP,  
symbol SYMBOL,  
price DOUBLE,  
amount DOUBLE  
) TIMESTAMP(timestamp)  #Designated timestamp value
  PARTITION BY DAY; # Chosen partition resolution 

```

Dense tables are preferred, better to separate tables if sparse. 

Time to live period (TTL) set period after which eligible for auto removal. 

```sql
CREATE TABLE trades (  
timestamp TIMESTAMP,  
symbol SYMBOL,  
price DOUBLE,  
amount DOUBLE  
) TIMESTAMP(timestamp)  
PARTITION BY DAY  
TTL 1 WEEK;
```



# Types

Symbols for categorical data.
- Use symbols for **categorical data** with a limited number of unique values (e.g., country codes, stock tickers, factory floor IDs).
- Symbols are fine for **storing up to a few million distinct values** but should be avoided beyond that.
- Avoid using a **`SYMBOL`** for columns that would be considered a `PRIMARY KEY` in other databases.
- **If very high cardinality is expected**, use **`VARCHAR`** instead of **`SYMBOL`**.
- **Symbols are compact on disk**, reducing storage overhead.
- **Symbol capacity defaults to 256**, but it will dynamically expand as needed, causing temporary slowdowns.
- **If you expect high cardinality, define the symbol capacity at table creation time** to avoid performance issues.

The **`TIMESTAMP`** type is recommended over **`DATETIME`**, unless you have checked the data types reference and you know what you are doing.

Use **`VARCHAR`** instead for general string storage.

QuestDB has a dedicated **[`UUID`](https://questdb.com/blog/uuid-coordination-free-unique-keys/)** type, which is more efficient than storing UUIDs as `VARCHAR`.

# Other

- QuestDB **does not enforce** `PRIMARY KEYS`, `FOREIGN KEYS`, or **`NOT NULL`** constraints.
- **Joins between tables work even without referential integrity**, as long as the data types on the [join condition](https://questdb.com/docs/reference/sql/join/) are compatible.
- **[Duplicate data](https://questdb.com/docs/concept/deduplication/) is allowed by default**, but `UPSERT KEYS` can be defined to **ensure uniqueness**.
- **Deduplication in QuestDB happens on an exact timestamp and optionally a set of other columns (`UPSERT KEYS`)**.
- **Deduplication has no noticeable performance penalty**.

