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

```
"host":"localhost",
"port":"8812",
"user":"admin",
"pwd":"quest",
"dbname":"qdb"
```

# Schema

