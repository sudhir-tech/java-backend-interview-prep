# System Design — File 07: SQL vs NoSQL & Choosing the Right Database

A common system-design interview question is:

> "Why did you choose SQL instead of NoSQL?"

A strong answer is not:

```text
SQL is better.
```

or:

```text
NoSQL scales better.
```

The correct answer is:

> **Choose the data store based on access patterns, consistency, relationships, scale, latency, and operational requirements.**

---

# 1. SQL vs NoSQL

The broad distinction is:

```text
SQL
→ relational data model

NoSQL
→ several non-relational data models
```

NoSQL is not one single type of database.

Common NoSQL categories include:

```text
Key-value
Document
Wide-column
Graph
```

---

# 2. SQL Databases

Examples:

```text
MySQL
PostgreSQL
Oracle
SQL Server
```

Typical characteristics:

```text
Tables
Rows
Columns
Relationships
SQL queries
Transactions
Constraints
Indexes
```

---

# 3. NoSQL Databases

Examples:

```text
Redis       → key-value / data structures
MongoDB     → document
Cassandra   → wide-column
DynamoDB    → key-value/document
Neo4j       → graph
```

Each solves different problems.

---

# 4. Don't Think "SQL vs NoSQL"

Instead think:

```text
What problem am I solving?
        ↓
What access pattern do I have?
        ↓
What guarantees do I need?
        ↓
Which database fits?
```

This mindset is much better for interviews.

---

# 5. SQL Strengths

Relational databases are strong when you need:

```text
Structured schema
Relationships
Transactions
Constraints
Complex queries
Joins
Strong consistency
```

Examples:

```text
Orders
Payments
Inventory
Banking
Accounting
User management
```

---

# 6. SQL Weaknesses

Potential challenges at very large scale:

```text
Vertical scaling limits
Complex sharding
High write throughput
Very large datasets
Schema changes
```

Modern relational databases can scale very far, so don't assume they fail at large scale.

---

# 7. NoSQL Strengths

Depending on the database, NoSQL systems can provide:

```text
Flexible schemas
High throughput
Horizontal scaling
Large distributed datasets
Low-latency access
Specialized access patterns
```

But these are not universal properties of every NoSQL database.

---

# 8. NoSQL Trade-offs

Potential trade-offs include:

```text
Limited joins
Application-managed relationships
Different transaction semantics
Query restrictions
Consistency trade-offs
Operational complexity
Data duplication
```

Again, the exact behavior depends on the specific database.

---

# 9. Key-Value Databases

Model:

```text
Key → Value
```

Example:

```text
user:101 → {...}
```

Very fast for:

```text
Lookup by key
Session data
Caching
Counters
Feature flags
```

Examples:

```text
Redis
DynamoDB
```

---

# 10. Document Databases

Store JSON-like documents.

Example:

```json
{
  "id": 101,
  "name": "Laptop",
  "price": 65000,
  "tags": ["electronics", "computer"]
}
```

Good for:

```text
Flexible records
Nested data
Rapid schema evolution
Document-centric access
```

Example:

```text
MongoDB
```

---

# 11. Wide-Column Databases

Data is organized around partitions and columns rather than traditional relational tables.

Examples:

```text
Cassandra
ScyllaDB
```

Useful for workloads requiring:

```text
High write throughput
Large distributed datasets
Predictable access patterns
Horizontal scaling
```

---

# 12. Graph Databases

Model:

```text
Nodes
Edges
Properties
```

Example:

```text
User
  ↓ follows
User
  ↓ likes
Product
```

Good for:

```text
Social networks
Recommendation relationships
Fraud networks
Dependency graphs
Knowledge graphs
```

Example:

```text
Neo4j
```

---

# 13. SQL Example — E-commerce

A relational design:

```text
users
products
orders
order_items
payments
inventory
```

Relationships:

```text
users
  ↓
orders
  ↓
order_items
  ↓
products
```

This is a natural SQL workload.

---

# 14. Document Example — Product Catalog

A product can have different attributes.

Laptop:

```json
{
  "id": 1,
  "name": "Laptop",
  "ram": "16GB",
  "storage": "1TB"
}
```

Phone:

```json
{
  "id": 2,
  "name": "Phone",
  "camera": "50MP",
  "battery": "5000mAh"
}
```

A document model can represent these varying attributes naturally.

---

# 15. Key-Value Example — Sessions

```text
session:abc123
    ↓
{
   userId: 42,
   expiresAt: ...
}
```

The primary access pattern is:

```text
Get by session ID
```

A key-value store is a natural fit.

---

# 16. Graph Example — Recommendations

Suppose:

```text
User A
  ↓ purchased
Product X

User B
  ↓ purchased
Product X

User B
  ↓ purchased
Product Y
```

A graph can make relationship traversal natural.

---

# 17. The Most Important Question: Access Pattern

Suppose the requirement is:

```text
Get user by ID
```

Key-value is excellent.

Suppose:

```text
Find orders by user,
filter by status,
sort by date,
join payment information.
```

A relational database may be a better fit.

---

# 18. Schema Flexibility

SQL generally uses a defined schema.

Example:

```text
Product
----------------
id
name
price
category_id
```

Document databases can allow documents with different fields.

This can be useful when:

```text
Attributes vary significantly
Schema changes frequently
```

But flexible schema doesn't mean:

```text
No schema needed.
```

You still need application-level data contracts.

---

# 19. Transactions

Relational databases have mature transaction support.

Example:

```text
Create Order
+
Create Order Items
+
Reserve Inventory
```

These may need transactional coordination.

Some NoSQL databases also support transactions, but their capabilities, performance characteristics and scope vary.

Don't claim:

```text
NoSQL doesn't support transactions.
```

That's incorrect.

---

# 20. Strong Consistency

For:

```text
Payment
Inventory
Account balance
```

you may need strong consistency or carefully designed concurrency control.

A database supporting the required consistency model is important.

---

# 21. Eventual Consistency

Eventual consistency means replicas or derived views may temporarily differ but converge when updates propagate.

Example:

```text
Primary:
price = 600

Replica:
price = 500
```

After replication:

```text
Replica:
price = 600
```

This can be acceptable for:

```text
Analytics
Recommendations
Some search indexes
Some social counters
```

when business requirements permit it.

---

# 22. CAP Theorem

CAP is frequently discussed in distributed database design.

It describes a trade-off among:

```text
Consistency
Availability
Partition tolerance
```

during a network partition.

---

# 23. Consistency

In simplified terms:

> A read sees data consistent with the system's chosen consistency guarantees.

Don't reduce consistency to:

```text
"Every server always has exactly the same value."
```

Distributed systems have multiple consistency models.

---

# 24. Availability

A system continues responding to requests.

Again, availability is about whether operations can be served, not whether every response necessarily contains the newest possible data.

---

# 25. Partition Tolerance

A distributed system must continue operating despite communication failures between nodes.

Network partitions can happen.

Therefore, in a distributed system:

```text
Partition tolerance
```

is generally unavoidable.

The practical question becomes:

```text
What consistency/availability behavior do we choose during a partition?
```

---

# 26. CAP Interview Answer

A concise answer:

> "CAP says that when a distributed system experiences a network partition, it has to trade off between consistency and availability. Partition tolerance is important because network partitions are unavoidable in distributed systems."

---

# 27. CAP Does NOT Mean

It does not simply mean:

```text
Pick exactly two forever.
```

The trade-off becomes especially relevant when a partition occurs.

This is an important interview nuance.

---

# 28. ACID vs BASE

### ACID

```text
Atomicity
Consistency
Isolation
Durability
```

Commonly associated with relational transactional systems.

### BASE

Often used to describe systems emphasizing:

```text
Basically Available
Soft state
Eventual consistency
```

These are conceptual models, not strict classifications of every database.

---

# 29. SQL vs NoSQL Decision Matrix

| Requirement | Often a good fit |
|---|---|
| Complex joins | SQL |
| Strong transactions | SQL |
| Structured relational data | SQL |
| Simple key lookup | Key-value |
| Flexible document model | Document DB |
| Huge distributed write workload | Wide-column |
| Relationship traversal | Graph |
| Ultra-fast cache/session | Redis |

This is a starting point, not a universal rule.

---

# 30. E-commerce Database Choice

For your e-commerce backend, a sensible starting point is:

```text
MySQL
```

because you have:

```text
Users
Products
Orders
Order Items
Payments
Inventory
```

and relationships plus transactional behavior matter.

---

# 31. Add Redis

For frequently accessed data:

```text
MySQL
  ↓
Source of truth

Redis
  ↓
Cache
```

This is usually better than replacing MySQL with Redis.

---

# 32. Add Search

If product search becomes complex:

```text
Product Service
      ↓
MySQL
      ↓
Search Index
```

A search engine such as:

```text
Elasticsearch
OpenSearch
```

can support:

```text
Full-text search
Filtering
Ranking
Facets
```

The relational database can remain the source of truth.

---

# 33. Polyglot Persistence

Using different databases for different workloads is called:

```text
Polyglot persistence
```

Example:

```text
MySQL
→ transactional data

Redis
→ cache

Elasticsearch
→ search

Kafka
→ event streaming
```

This can be powerful.

But it also increases:

```text
Operational complexity
Data synchronization
Monitoring
Failure scenarios
```

---

# 34. Don't Use NoSQL Just Because It Scales

This is a common interview mistake.

Bad answer:

> "We'll use MongoDB because it scales horizontally."

Better:

> "The access pattern is document-oriented and doesn't require complex relational joins, so a document database may simplify the data model and horizontal scaling."

Explain the workload.

---

# 35. Don't Use SQL Just Because You Know It

The opposite mistake is:

> "I always use MySQL."

Instead ask:

```text
What queries?
What consistency?
What scale?
What relationships?
What latency?
```

Then decide.

---

# 36. Database-per-Workload

A large system may use:

```text
Transactional DB
Search DB
Cache
Analytics warehouse
Object storage
Event stream
```

Each system solves a different problem.

---

# 37. OLTP

OLTP:

```text
Online Transaction Processing
```

Typical workloads:

```text
Create order
Update inventory
Process payment
Update user profile
```

Characteristics:

```text
Many small transactions
Low latency
Strong correctness requirements
```

Relational databases are common here.

---

# 38. OLAP

OLAP:

```text
Online Analytical Processing
```

Examples:

```text
Monthly sales analysis
Customer trends
Business dashboards
Large aggregations
```

Usually involves:

```text
Large scans
Aggregations
Historical data
```

Analytics warehouses are commonly used.

---

# 39. Don't Run Analytics on the Primary OLTP Database

Suppose:

```text
Production MySQL
```

is serving:

```text
Checkout
Orders
Payments
```

Then someone runs:

```sql
SELECT ...
FROM orders
GROUP BY ...
```

over hundreds of millions of rows.

Potential result:

```text
Production workload slows
```

Separate analytics workloads where necessary.

---

# 40. CQRS Preview

CQRS:

```text
Command Query Responsibility Segregation
```

Separates:

```text
Write model
Read model
```

Example:

```text
Commands
   ↓
Transactional DB

Queries
   ↓
Optimized read model
```

Useful when:

```text
Read and write workloads are very different.
```

But it adds complexity.

---

# 41. Read Model Example

Orders are stored transactionally:

```text
MySQL
```

A separate read model could be optimized for:

```text
Order history
Dashboard
Search
Reporting
```

Updates can be propagated using events.

---

# 42. Database Selection Checklist

Ask:

```text
1. What are the entities?
2. What are the access patterns?
3. Do we need joins?
4. Do we need transactions?
5. What consistency is required?
6. What is the data volume?
7. What is the read/write ratio?
8. What latency is required?
9. How does the data grow?
10. How will we scale it?
```

---

# 43. SQL Selection Checklist

Choose SQL when you need:

```text
Relationships
Transactions
Constraints
Complex queries
Strong consistency
Mature reporting/querying
```

---

# 44. NoSQL Selection Checklist

Consider NoSQL when you need:

```text
Specific high-scale access patterns
Flexible documents
Very large distributed workloads
Simple key-based access
Specialized data models
```

---

# 45. Key-Value Selection Checklist

Consider key-value when:

```text
Access is primarily by key
Very low latency matters
Relationships are minimal
```

Examples:

```text
Sessions
Cache
Counters
Simple user preferences
```

---

# 46. Document Selection Checklist

Consider document DB when:

```text
Data is naturally document-shaped
Nested objects are common
Schema varies
Queries are primarily within a document
```

---

# 47. Wide-Column Selection Checklist

Consider wide-column systems when:

```text
Very large scale
High write throughput
Predictable partition-key queries
Distributed storage
```

---

# 48. Graph Selection Checklist

Consider graph DB when:

```text
Relationships are the primary query
Traversal depth matters
Connections are more important than tabular aggregation
```

---

# 49. Search Engine Is Not the Source of Truth by Default

A common architecture:

```text
MySQL
  ↓
ProductUpdated event
  ↓
Search Index
```

Search data can be rebuilt from authoritative data if designed properly.

This reduces dependency on the search engine as the canonical store.

---

# 50. Object Storage

Files such as:

```text
Images
Videos
PDFs
Backups
```

usually belong in object storage rather than a relational database.

Examples:

```text
Amazon S3
Azure Blob Storage
Google Cloud Storage
```

The database can store:

```text
Object URL/key
Metadata
Ownership
Permissions
```

---

# 51. Database + Object Storage

Example:

```text
Product
---------
id
name
image_key
```

Actual image:

```text
Object Storage
```

This avoids putting large binary files directly into the transactional database in many architectures.

---

# 52. Multi-Database Consistency

Suppose:

```text
MySQL
Redis
Elasticsearch
Kafka
```

all contain related information.

They won't automatically update atomically.

You need:

```text
Events
Retries
Idempotency
Reconciliation
Outbox pattern
```

depending on the architecture.

---

# 53. Eventual Consistency Example

Product update:

```text
MySQL
 ↓
Product updated
 ↓
Event
 ↓
Search index updated
 ↓
Redis invalidated
```

For a short period:

```text
MySQL = new
Search = old
Redis = old
```

If the business can tolerate this, eventual consistency is acceptable.

---

# 54. Strong Consistency Example

Payment:

```text
Payment status
```

You generally don't want:

```text
Payment = SUCCESS
```

in one place while another critical component believes:

```text
Payment = PENDING
```

for an unacceptable period.

The consistency requirements are much stricter.

---

# 55. Data Duplication in NoSQL

NoSQL systems often intentionally duplicate data to optimize reads.

Example:

```json
{
  "orderId": 101,
  "customer": {
    "id": 42,
    "name": "Sudhir"
  }
}
```

The customer name may also exist elsewhere.

This can reduce joins.

But updates become harder.

---

# 56. Denormalization in NoSQL

A useful mindset:

```text
SQL:
Normalize around entities.

NoSQL:
Often model around queries.
```

This is a simplification, but a useful interview mental model.

---

# 57. Access Pattern Example

Requirement:

```text
Get user's last 20 orders.
```

A relational model may query:

```text
orders
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20
```

A NoSQL model might store data specifically around:

```text
user_id
```

to make that access pattern efficient.

---

# 58. Hot Partition in NoSQL

Suppose partition key is:

```text
country
```

and:

```text
India = 80% traffic
```

Then one partition can become overloaded.

A good partition key distributes traffic.

---

# 59. Cardinality

Cardinality describes the number of distinct values.

For partition/shard keys, higher cardinality often provides more opportunities for distribution.

Example:

```text
country
```

has low cardinality.

```text
user_id
```

has much higher cardinality.

But high cardinality alone doesn't guarantee a good distribution.

---

# 60. Time-Based Partitioning

For logs/events:

```text
2026-01
2026-02
2026-03
```

can be useful.

But if almost all writes go to:

```text
current month
```

the current partition can become hot.

This is why partition design must consider both:

```text
Storage distribution
Traffic distribution
```

---

# 61. Consistency vs Availability Example

Imagine two replicas:

```text
Node A:
inventory = 0

Node B:
inventory = 1
```

A network partition occurs.

If the system continues accepting writes independently:

```text
Availability increases
```

but conflicting values can occur.

If the system refuses operations until consistency can be guaranteed:

```text
Consistency increases
```

but availability decreases.

This illustrates the CAP trade-off.

---

# 62. Database Migration

Changing databases is not just:

```text
Export
Import
Done
```

Large migrations require:

```text
Schema compatibility
Dual writes or CDC
Backfill
Validation
Cutover
Rollback
Monitoring
```

---

# 63. Schema Migration

For production systems, prefer backward-compatible changes where possible.

Example:

```text
1. Add new nullable column
2. Deploy code that understands both versions
3. Backfill data
4. Start writing new field
5. Remove old field later
```

This helps rolling deployments.

---

# 64. Database Versioning

Use migration tools such as:

```text
Flyway
Liquibase
```

to manage schema changes consistently.

---

# 65. Backup vs Replication

Important distinction:

```text
Replication
→ Availability / read scaling

Backup
→ Recovery from data loss/corruption
```

Replication does not replace backups.

If bad data is replicated:

```text
Primary bad data
 ↓
Replica bad data
```

Both can be wrong.

---

# 66. Disaster Recovery

For critical systems consider:

```text
Backups
Replication
Multi-zone
Multi-region
Restore testing
RPO
RTO
```

The appropriate level depends on business requirements.

---

# 67. Cost

Database decisions also affect cost.

Consider:

```text
Compute
Memory
Storage
IOPS
Network
Replicas
Backups
Operations
```

A technically elegant system that costs 20× more may not be the right system.

---

# 68. Interview Question

### When would you choose MongoDB over MySQL?

Answer:

> "I'd consider MongoDB when the data is naturally document-oriented, schema flexibility is valuable and the primary access patterns can be served without heavy relational joins. I wouldn't choose it simply because the system needs scale."

---

# 69. Interview Question

### When would you choose Cassandra?

Answer:

> "I'd consider Cassandra for very large distributed workloads with high write throughput and predictable query patterns, especially when horizontal scaling and availability across nodes are important."

---

# 70. Interview Question

### When would you choose Redis?

Answer:

> "I'd primarily use Redis for low-latency key-based access such as caching, sessions, counters and some coordination or rate-limiting use cases."

---

# 71. Interview Question

### When would you choose a graph database?

Answer:

> "I'd consider one when relationships and graph traversal are central to the workload, such as social graphs, recommendation relationships or dependency analysis."

---

# 72. Interview Question

### Can SQL scale horizontally?

Answer:

> "Yes. Relational databases can scale through read replicas, partitioning, sharding and distributed SQL systems. The point isn't that SQL can't scale; it's that some workloads make horizontal scaling more complex."

---

# 73. Interview Question

### Is NoSQL always faster?

Answer:

> "No. Performance depends on the workload, schema, query pattern, indexing, hardware and implementation. A well-designed SQL query can be much faster than an unsuitable NoSQL design."

---

# 74. Interview Question

### Does NoSQL mean no schema?

Answer:

> "No. Many NoSQL databases provide flexible schemas, but the application still needs a clear data model and validation rules."

---

# 75. Interview Question

### Why use multiple databases?

Answer:

> "Different workloads may have different requirements. For example, MySQL can handle transactional data, Redis can provide caching, and a search engine can handle full-text search. This is polyglot persistence, but it increases operational and consistency complexity."

---

# 76. Interview Question

### What is polyglot persistence?

Answer:

> "It's using different data stores for different workloads instead of forcing every use case into one database."

---

# 77. Interview Question

### What is CQRS?

Answer:

> "CQRS separates the write model from the read model so each can be optimized for its workload. It can be useful at scale but introduces additional synchronization and operational complexity."

---

# 78. Practical Scenario

### Requirement:

```text
Bank transfer
```

Need:

```text
Strong correctness
Transactions
Consistency
Auditability
```

Likely starting point:

```text
Relational database
```

---

# 79. Practical Scenario

### Requirement:

```text
Session storage
```

Access:

```text
Get session by ID
```

Likely fit:

```text
Redis
```

---

# 80. Practical Scenario

### Requirement:

```text
Product search
```

Features:

```text
Full text
Filters
Ranking
Facets
```

Likely architecture:

```text
MySQL → Source of truth
Search engine → Search index
```

---

# 81. Practical Scenario

### Requirement:

```text
Social network relationship traversal
```

Queries:

```text
Friends of friends
Shortest relationship path
People connected to X
```

A graph database may be worth evaluating.

---

# 82. Practical Scenario

### Requirement:

```text
Billions of time-series events
Very high write throughput
Known query patterns
```

A distributed wide-column/time-series system may be more appropriate than a traditional single-node relational design.

---

# 83. Practical Scenario

### Requirement:

```text
Flexible product attributes
Different fields per product category
Document-oriented reads
```

A document database may be a candidate.

But if the same system also needs:

```text
Complex order/payment transactions
```

you might still keep those in a relational database.

---

# 84. The "Right Database" Can Be More Than One

A mature architecture may be:

```text
                 Application
                      |
        +-------------+-------------+
        |             |             |
       SQL          Redis        Search
        |             |             |
 Transactions      Cache       Full-text
```

The architecture should be driven by requirements.

---

# 85. Database Choice Framework

Use this in interviews:

```text
Data model
    ↓
Access pattern
    ↓
Consistency
    ↓
Transaction requirements
    ↓
Scale
    ↓
Latency
    ↓
Availability
    ↓
Operational complexity
    ↓
Cost
```

Then choose.

---

# 86. Final Comparison

| Feature | SQL | Key-Value | Document | Wide-Column | Graph |
|---|---|---|---|---|---|
| Relationships | Excellent | Limited | Limited/embedded | Limited | Excellent |
| Joins | Strong | No | Usually limited | Limited | Traversal |
| Transactions | Strong | Varies | Varies | Varies | Varies |
| Schema flexibility | Lower | High | High | High | High |
| Key lookup | Good | Excellent | Excellent | Excellent | Good |
| Horizontal scale | Possible | Strong | Strong | Strong | Depends |
| Best for | Transactions | Fast lookup | Documents | Huge distributed workloads | Relationships |

Always verify the exact guarantees of the specific product.

---

# 87. Final Checklist

You should be able to explain:

```text
□ SQL
□ NoSQL
□ Key-value
□ Document
□ Wide-column
□ Graph
□ SQL transactions
□ Eventual consistency
□ CAP theorem
□ ACID vs BASE
□ Access-pattern-driven design
□ Polyglot persistence
□ OLTP
□ OLAP
□ CQRS
□ Search indexes
□ Object storage
□ Data duplication
□ Hot partitions
□ Cardinality
□ Database migration
□ Schema migration
□ Backup vs replication
□ RPO/RTO
□ Cost considerations
```

---

# 88. One-Minute Interview Answer

### "How would you choose between SQL and NoSQL?"

> "I'd start with the access patterns and consistency requirements rather than choosing based on popularity. If the system has strong relationships, transactions and complex queries, I'd generally start with a relational database such as MySQL or PostgreSQL. If the workload is primarily key-based, document-oriented or requires a specialized distributed data model, I'd consider an appropriate NoSQL store. In a larger system, I may use multiple databases for different workloads, but I'd account for the additional consistency and operational complexity."

---

# 89. Key Takeaway

> **There is no universally best database. The right database is the one whose data model, consistency guarantees, query capabilities and scaling characteristics match the workload.**

Think:

```text
Requirements
     ↓
Access patterns
     ↓
Data model
     ↓
Consistency
     ↓
Scale
     ↓
Database choice
```

**File 07 complete.**
