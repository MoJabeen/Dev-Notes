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
- 