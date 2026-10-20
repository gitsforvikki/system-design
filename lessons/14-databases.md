# Lesson 14 — Databases ⭐⭐⭐⭐⭐

> **Goal:** Learn database architecture for real system design: relational and document models, transactions, concurrency, indexes, performance, scaling, and practical Node.js examples. Explanations use simple English and concrete applications.

## 01 — Why databases exist

An application needs data to survive process restarts and to be shared between users and servers. A database manages storage, queries, concurrency, and recovery. CareerLoop stores applications; CodeBuddy stores users/messages; ShopHub stores products/orders. A plain JavaScript array is not a durable multi-user database.

## 02 — Database vs DBMS

A database is the organized stored data; a database management system (DBMS) is the software managing it, such as PostgreSQL or MongoDB. Your API talks to the DBMS through a driver/ORM/ODM. The DBMS handles indexing, concurrency, permissions, recovery, and storage.

## 03 — End-to-end request architecture

```mermaid
flowchart LR
  Browser --> API[Next.js / Express API]
  API --> Pool[Connection pool]
  Pool --> DB[(PostgreSQL)]
  DB --> Disk[Durable storage]
  API --> Cache[(Optional Redis cache)]

Redis can speed up reads, but the authoritative database decides correctness for important writes.
```

## 04 — Database selection starts with access patterns

Ask: What entities exist? How are they related? Which reads and writes happen most often? How many concurrent users? Which queries need sorting/filtering? Are multi-record transactions necessary? What consistency, uptime, recovery, and cost are required? Pick technology after understanding these needs.

## 05 — Relational model

SQL databases represent structured data in tables of rows and columns. A users table can be related to applications by a user_id foreign key. The database can enforce relationships and uniqueness, rather than relying entirely on application code.

## 06 — SQL example for CareerLoop

~~~sql
CREATE TABLE users (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email TEXT NOT NULL UNIQUE
);
CREATE TABLE applications (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id),
  company TEXT NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('Applied','Screening','Interview','Offer','Rejected')),
  applied_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
~~~
This is a teaching schema; production design may use UUIDs, separate company tables, audit records, and additional constraints.

## 07 — Primary keys and foreign keys

A primary key uniquely identifies each row and cannot be NULL. A foreign key checks that referenced rows exist (subject to constraint configuration). A foreign key does not automatically create an index on the referencing column in PostgreSQL; add indexes based on query patterns.

## 08 — SQL joins

~~~sql
SELECT a.id, u.email, a.company
FROM applications AS a
JOIN users AS u ON u.id = a.user_id
WHERE a.status = 'Interview';
~~~
INNER JOIN returns matching pairs. LEFT JOIN retains left-side rows without matches. Joins make normalized relationships queryable, but join cost depends on cardinality, indexes, and plans.

## 09 — NoSQL is a category, not one data model

NoSQL includes document stores (MongoDB), key/value stores (Redis), wide-column systems, and graph databases. These systems differ significantly. NoSQL does not mean “no schema,” “no transactions,” or “always faster.” Choose by workload and guarantees.

## 10 — Document database model

MongoDB stores BSON documents in collections. Example: a public CodeBuddy profile document can contain _id, name, skills, and an embedded preferences object. Flexible document shapes simplify some aggregate reads, but validation and indexes remain necessary.

## 11 — MongoDB example

~~~js
// MongoDB shell
 db.users.insertOne({
   name: "Asha",
   skills: ["React", "Node.js"],
   location: { city: "Bengaluru" }
 });
 db.users.find({ skills: "React" });
~~~
MongoDB can use schema validation; Mongoose adds application-side schemas, casting, middleware, and model APIs. Neither removes the need for thoughtful data modeling.

## 12 — Embedding vs referencing

Embedding keeps related data in one document and can reduce joins/lookups for bounded child collections. Referencing stores IDs and separates independently growing entities. Example: user preferences may embed inside user; CodeBuddy chat messages should generally be separate documents instead of an unbounded messages array inside a conversation document.

## 13 — SQL vs MongoDB decision table

| Question | PostgreSQL often fits | MongoDB often fits |
|---|---|---|
| Many related entities and joins | Strong choice | Possible with references and $lookup |
| Flexible aggregate documents | Supported with JSONB | Natural document model |
| Complex relational constraints | Strong built-in support | Often application-level modeling |
| Transactions | Mature multi-row ACID | Supports multi-document transactions |
| Query flexibility | Rich SQL | Rich document query/aggregation |
| Scale | Depends on architecture | Depends on architecture |
Neither is universally faster. Measure actual workload.

## 14 — OLTP vs OLAP

OLTP (online transaction processing) handles frequent small reads/writes such as placing an order. OLAP (online analytical processing) handles large scans and aggregations such as quarterly revenue by category. Running heavy analytics on a primary transactional database may affect user latency; consider replicas, warehouses, or ETL/ELT pipelines when justified.

## 15 — ACID overview

Atomicity: all-or-nothing transaction. Consistency: constraints/invariants remain valid under valid transactions. Isolation: concurrent transactions behave according to a specified isolation level. Durability: committed changes survive failures according to system guarantees. ACID is not a blanket promise that application logic has no bugs.

## 16 — Atomicity example

A ShopHub order creation may need to insert order, reserve inventory, and record payment intent together when they share one database. If one step fails, roll back the transaction. External payment gateway calls are not automatically rolled back by a database transaction; use idempotency, reconciliation, and saga/outbox patterns where appropriate.

## 17 — PostgreSQL transaction example

~~~sql
BEGIN;
UPDATE inventory SET quantity = quantity - 1
WHERE product_id = 42 AND quantity > 0;
-- Application must verify exactly one row was updated.
INSERT INTO orders (user_id, status) VALUES (7, 'CREATED');
COMMIT;
~~~
If the UPDATE affects zero rows, the application should ROLLBACK instead of creating the order. Keep transactions short and handle errors correctly.

## 18 — Isolation anomalies

Dirty read: see uncommitted data. Non-repeatable read: re-read same row and see a changed value. Phantom: re-run a predicate and see a changed set of rows. Lost update: concurrent writes overwrite each other. Write skew: two concurrent transactions each observe a valid snapshot then together violate a business rule. Different isolation levels prevent different anomalies.

## 19 — Isolation levels

SQL standard names: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE. Implementations differ. PostgreSQL treats READ UNCOMMITTED as READ COMMITTED and uses MVCC for snapshot-based behavior. SERIALIZABLE may abort transactions with serialization failures; the application must retry the entire transaction safely.

## 20 — MVCC intuition

Multi-version concurrency control keeps row versions so readers and writers can proceed with less blocking under supported isolation rules. Think of readers observing a suitable snapshot while writers create newer versions. MVCC reduces some contention but does not remove locks, conflicts, vacuum requirements, or transaction anomalies.

## 21 — Optimistic vs pessimistic concurrency

Optimistic: update only if version matches; retry or reject on conflict. Pessimistic: lock relevant rows (e.g. SELECT ... FOR UPDATE) before changing them. Optimistic suits infrequent conflicts; pessimistic can help high-contention critical operations but risks waiting/deadlocks.

## 22 — Lost update prevention

~~~sql
UPDATE products
SET stock = stock - 1
WHERE id = 42 AND stock > 0
RETURNING stock;
~~~
This conditional atomic update avoids read-then-write overselling for a single row. Multi-row invariants require appropriate transaction design.

## 23 — Normalization

Normalization reduces unnecessary duplication and update anomalies. First normal form uses atomic column values for the chosen relational model; second/third normal forms address certain dependencies between attributes and keys. Example: store company name once in companies rather than repeating it in every application when consistency matters. Normalization is a design tool, not a requirement to split every piece of data.

## 24 — Denormalization

Duplicate or precompute selected values to accelerate reads, e.g. company_name_snapshot on an application, or a dashboard count. Trade-off: more storage and update/invalidation complexity. Denormalize only for a measured query requirement and define how copies stay consistent.

## 25 — Index fundamentals

An index is an auxiliary data structure that helps locate rows without scanning every row. PostgreSQL commonly uses B-tree indexes; other types include GIN, GiST, BRIN, and hash. Indexes speed suitable reads but consume storage and slow writes because indexes must be maintained.

## 26 — B-tree mental model

```mermaid
flowchart TD
  Root[Root keys] --> L[Left index pages]
  Root --> R[Right index pages]
  L --> RowsA[Matching row locations]
  R --> RowsB[Matching row locations]

A B-tree narrows the search by ordered keys; the DB may still need to fetch table rows. The diagram is conceptual, not a literal on-disk layout.
```

## 27 — Single-column and composite indexes

~~~sql
CREATE INDEX applications_user_id_idx ON applications(user_id);
CREATE INDEX applications_user_status_applied_idx
ON applications(user_id, status, applied_at DESC);
~~~
The second index is useful for queries filtering by user and status and ordering by applied_at. Column order matters; don't assume it optimizes every permutation of predicates.

## 28 — Selectivity and cardinality

High-cardinality columns (many distinct values) can make indexes valuable for selective lookups. A low-cardinality status column alone may be less selective, though combined indexes, partial indexes, and query patterns can change the answer. Planners estimate row counts and costs, not just whether an index exists.

## 29 — Covering and partial indexes

A covering index can satisfy needed columns without visiting the table in some cases (visibility rules still matter). Partial index stores only rows matching a predicate:

~~~sql
CREATE INDEX applications_active_idx
ON applications(user_id, applied_at DESC)
WHERE status IN ('Applied','Screening','Interview');
~~~
Choose predicates matching real queries; too many indexes hurt writes.

## 30 — EXPLAIN and query plans

~~~sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM applications
WHERE user_id = 7 AND status = 'Interview'
ORDER BY applied_at DESC LIMIT 20;
~~~
EXPLAIN shows the chosen plan; ANALYZE executes the query, so be careful with modifying statements. Read scan type, estimated vs actual rows, loops, timing, and buffer usage. Sequential scan is not automatically bad for small tables.

## 31 — N+1 queries

N+1 happens when an API fetches a list and then performs one query per item for related data. Example: fetch 100 applications then 100 company queries. Solve with joins, batched IN queries, ORM eager loading, or carefully designed document embedding. Watch result explosion when joining multiple one-to-many relations.

## 32 — Pagination

OFFSET/LIMIT is simple but deep offsets can become expensive and changing datasets can cause skipped/duplicate items. Keyset pagination uses a stable sort tuple, such as (applied_at, id), and a cursor. Include a unique tie-breaker and matching index. Cursor design must handle direction, filtering, and updates.

## 33 — Keyset pagination example

~~~sql
SELECT id, company, applied_at
FROM applications
WHERE user_id = 7
  AND (applied_at, id) < ('2026-10-01T10:00:00Z', 500)
ORDER BY applied_at DESC, id DESC
LIMIT 20;
~~~
An index on (user_id, applied_at DESC, id DESC) supports this access pattern. The cursor values come from the previous page's last row.

## 34 — Connection lifecycle

Opening a DB connection often involves network setup, authentication, and resource allocation. Creating one per HTTP request can overload DB connection limits. A pool reuses a bounded number of connections; requests wait for available slots. Pool size must account for total application instances, not just one process.

## 35 — Pool sizing example

If 20 Node.js instances each allow 10 connections, the theoretical maximum is 200 database connections, before admin jobs or other services. A database with a lower connection limit can fail even if each instance seems modest. Serverless scaling can magnify this; use suitable pooling/proxies and workload limits.

## 36 — Node.js PostgreSQL example

~~~js
import pg from "pg";
const pool = new pg.Pool({ connectionString: process.env.DATABASE_URL, max: 10 });

export async function getApplications(userId) {
  const result = await pool.query(
    "SELECT id, company, status FROM applications WHERE user_id = $1",
    [userId]
  );
  return result.rows;
}
~~~
Use parameterized queries, not string concatenation. Reuse the pool; shut it down gracefully. For a transaction, acquire one client and run BEGIN/COMMIT/ROLLBACK on that same client.

## 37 — Node.js transaction example

~~~js
const client = await pool.connect();
try {
  await client.query("BEGIN");
  const updated = await client.query(
    "UPDATE inventory SET quantity = quantity - 1 WHERE product_id = $1 AND quantity > 0 RETURNING product_id",
    [productId]
  );
  if (updated.rowCount !== 1) throw new Error("Out of stock");
  await client.query("INSERT INTO orders(user_id, status) VALUES ($1, $2)", [userId, "CREATED"]);
  await client.query("COMMIT");
} catch (error) {
  await client.query("ROLLBACK");
  throw error;
} finally {
  client.release();
}
~~~
Assumes inventory/orders tables exist and userId/productId are validated. Real order design needs product/quantity/price details and payment coordination.

## 38 — ORM vs ODM vs raw driver

Raw driver: direct DB operations. ORM such as Drizzle/Prisma maps application code to relational queries and migrations. ODM such as Mongoose models MongoDB documents, validation, hooks, and queries. Abstractions improve ergonomics but don't replace knowledge of indexes, transactions, query plans, or database semantics.

## 39 — Migrations

A schema migration changes tables/indexes/constraints in a controlled, versioned way. Production deploys should consider backward compatibility: add nullable/new columns first, deploy compatible code, backfill, then enforce constraints/remove old columns. Large table rewrites and index builds may lock or load the database; choose online/concurrent operations where supported.

## 40 — Backups and disaster recovery

Backups protect against deletion, corruption, and operator mistakes. Replicas primarily help availability and read capacity; they can replicate accidental deletes too. Define RPO (maximum tolerable data loss) and RTO (maximum acceptable recovery time), test restore procedures, and protect backup credentials.

## 41 — Replication and sharding preview

Replication maintains copies of data, often for availability/read scaling. Sharding partitions data across nodes for capacity. Both introduce lag, routing, and consistency complexity. Lesson 15 will cover them in depth; neither should be your first response to an unindexed query.

## 42 — SQL injection and least privilege

Never concatenate untrusted input into SQL. Use parameterized queries, validate inputs, and restrict DB credentials to necessary operations. Separate application and migration/admin privileges. Encrypt transport and secrets; use audit logs and rotate credentials as appropriate.

## 43 — Database observability

Monitor p50/p95/p99 query latency, slow queries, lock waits, deadlocks, connection pool saturation, CPU, memory, disk I/O, storage growth, replication lag, errors, and transaction retries. Measure from both application and database perspectives. Fix expensive queries and indexes before assuming a new database is required.

## 44 — CareerLoop database design

PostgreSQL fits users, companies, applications, interviews, recruiters, and followups with relational integrity. Index application queries by user/status/date. Enforce user ownership in every query, not just UI filters. A Neon-managed PostgreSQL database still needs careful pooling, migrations, and monitoring.

## 45 — CodeBuddy database design

MongoDB can represent profiles and connection requests, while messages belong in separately indexed documents or collections. Index conversations by participants and messages by conversation/time. Avoid growing one unbounded array of messages inside a user document. Use unique indexes for relationships where appropriate.

## 46 — ShopHub database design

Products, orders, order items, payments, and inventory need clear invariants. An order should capture a snapshot of purchased prices rather than depending on mutable product prices later. Inventory decrements need atomic conditional updates or suitable transactions; payment gateway calls require idempotency/reconciliation beyond database ACID.

## 47 — Choosing SQL vs NoSQL in an interview

Explain entities and access patterns, then constraints and transactions, then query/index strategy, scaling, operational maturity, and trade-offs. Say why PostgreSQL or MongoDB is appropriate for this particular design; avoid “NoSQL is always faster” or “SQL cannot scale.”

## 48 — Common mistakes

No indexes on common filters; indexing every column; assuming foreign keys create indexes; keeping transactions open across slow external API calls; using OFFSET for massive deep pagination; leaking private rows across tenants; unbounded pool sizes; ignoring isolation anomalies; treating replicas as backups; choosing a database based only on trends.

## 49 — Interview Q&A

1. What is a DBMS? 2. SQL vs NoSQL? 3. What is a foreign key? 4. Embedding vs referencing? 5. What are ACID properties? 6. What is MVCC? 7. Isolation anomalies? 8. Optimistic vs pessimistic locking? 9. How do indexes work? 10. Why do indexes slow writes? 11. Composite index ordering? 12. What is N+1? 13. OFFSET vs keyset pagination? 14. Why use a connection pool? 15. Why do serverless connections need care? 16. What is normalization? 17. When denormalize? 18. OLTP vs OLAP? 19. Replication vs backup? 20. How to avoid SQL injection? 21. How to prevent overselling? 22. How do you choose a database for chat?

## 50 — Practice tasks

**A:** Design CareerLoop tables with constraints. **B:** Write joins for applications and companies. **C:** Add composite indexes and inspect EXPLAIN. **D:** Simulate two concurrent inventory purchases. **E:** Implement transaction rollback in Node.js. **F:** Design MongoDB chat messages with indexes. **G:** Replace N+1 queries with batching. **H:** Implement keyset pagination. **I:** Calculate total pool connections across instances. **J:** Explain your SQL/NoSQL choice and failure recovery in a 10-minute interview.

## 51 — Revision checklist

- [ ] I understand SQL and major NoSQL models.
- [ ] I can model primary/foreign keys and documents.
- [ ] I understand ACID, MVCC, and isolation levels.
- [ ] I can prevent lost updates and overselling.
- [ ] I can reason about normalization and denormalization.
- [ ] I can design indexes from query patterns.
- [ ] I can read EXPLAIN and identify N+1 queries.
- [ ] I understand connection pooling and migrations.
- [ ] I can explain backups, RPO/RTO, and security.
- [ ] I can justify DB choices for CareerLoop, CodeBuddy, ShopHub.

---

**Next: Lesson 15 — Replication & Sharding.** We'll cover primary/replica, consistency and lag, leader election, partitioning, shard keys, rebalancing, and production failure modes.
