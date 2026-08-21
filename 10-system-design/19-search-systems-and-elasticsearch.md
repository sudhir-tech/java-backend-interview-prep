# System Design — File 19: Search Systems & Elasticsearch

Search becomes important when users need to find information by keywords, phrases, filters, ranking, or partial matches.

Examples:

```text
Product search
Document search
Job search
Log search
User search
Autocomplete
```

A relational database can handle simple search, but dedicated search engines become valuable for large-scale, relevance-focused search.

---

## 1. Database Search vs Search Engine

Simple search:

```sql
SELECT *
FROM products
WHERE name LIKE '%phone%';
```

This can work for small datasets.

At scale, search requirements may include:

```text
Full-text search
Relevance ranking
Fuzzy matching
Autocomplete
Synonyms
Facets
Highlighting
Complex filters
```

A search engine is often a better fit.

---

## 2. Elasticsearch

Elasticsearch is a distributed search and analytics engine.

Conceptually:

```text
Application
    |
    v
Elasticsearch
    |
    +--> Search
    +--> Filtering
    +--> Ranking
    +--> Aggregations
```

It is commonly used for:

```text
Product search
Logs
Observability
Document search
Analytics
```

---

## 3. Elasticsearch Is Not Usually the Primary Database

A common architecture is:

```text
Application
   |
   +--> MySQL
   |      |
   |      +--> Source of truth
   |
   +--> Elasticsearch
          |
          +--> Search index
```

The database remains the authoritative source.

Elasticsearch is optimized for search.

---

## 4. Why Use a Search Index?

Searching directly in a large relational table can become expensive.

A search index creates structures optimized for:

```text
Text lookup
Ranking
Filtering
Aggregations
```

This improves search performance and functionality.

---

## 5. Inverted Index

The core concept behind full-text search is the inverted index.

Suppose documents contain:

```text
Doc 1 -> Java Spring Boot
Doc 2 -> Java Redis
Doc 3 -> Spring Redis
```

An inverted index might conceptually contain:

```text
java   -> Doc 1, Doc 2
spring -> Doc 1, Doc 3
redis  -> Doc 2, Doc 3
```

Instead of scanning every document, the search engine looks up terms in the index.

---

## 6. Tokenization

A text field can be broken into tokens.

Example:

```text
"Java Backend Developer"
```

could become:

```text
java
backend
developer
```

The analyzer determines how text is processed.

---

## 7. Analyzers

An Elasticsearch analyzer can perform steps such as:

```text
Character filtering
Tokenization
Lowercasing
Stop-word handling
Stemming
```

Example:

```text
"Running"
```

may be normalized depending on the analyzer.

---

## 8. Stemming

Stemming reduces related word forms to a common representation.

Example:

```text
running
runs
ran
```

Depending on the stemmer, related forms may map toward a common root.

Use carefully because aggressive stemming can produce unwanted matches.

---

## 9. Stop Words

Common words may add little search value.

Examples:

```text
the
a
an
is
of
```

A search analyzer may remove some stop words.

This depends on the application's search requirements.

---

## 10. Exact Match vs Full-Text Search

### Exact match

Useful for:

```text
status
category ID
product ID
country code
```

### Full-text search

Useful for:

```text
Product descriptions
Article content
Job descriptions
Documents
```

Don't treat every field as full-text.

---

## 11. Keyword vs Text Fields

A common Elasticsearch distinction:

```text
text
keyword
```

### text

Analyzed for full-text search.

Example:

```text
"Java Backend Developer"
```

### keyword

Stored as an exact value.

Example:

```text
"IN_STOCK"
```

A field may sometimes have both forms.

---

## 12. Multi-Fields

Example concept:

```text
name
  |
  +--> text
  |
  +--> keyword
```

This allows:

```text
Full-text search
+
Exact matching/sorting/aggregation
```

---

## 13. Query

A search query asks Elasticsearch for matching documents.

Conceptually:

```text
Search:
"java backend"
```

Elasticsearch analyzes the query and searches the index.

---

## 14. Match Query

A match query is commonly used for analyzed text.

Example:

```text
name matches "java backend"
```

The query text is analyzed according to the field's analyzer.

---

## 15. Term Query

A term query is intended for exact terms and is commonly used with keyword fields.

Example:

```text
status = "ACTIVE"
```

Do not use a term query blindly against analyzed text fields.

---

## 16. Boolean Queries

Search conditions can be combined:

```text
must
should
filter
must_not
```

Example:

```text
Search "phone"
AND
category = "electronics"
AND
price < 50000
```

---

## 17. Filter vs Query

A filter is useful for conditions where relevance scoring is not the main goal.

Example:

```text
category = electronics
price < 50000
inStock = true
```

Filters can often be cached and are generally more efficient for structured constraints.

---

## 18. Relevance Scoring

Search engines often rank results rather than simply returning matching rows.

Conceptually:

```text
Document A -> score 10.2
Document B -> score 7.5
Document C -> score 3.1
```

Higher score generally means a stronger match according to the query.

---

## 19. BM25

Elasticsearch commonly uses BM25 for text relevance scoring.

It considers factors such as:

```text
Term frequency
Inverse document frequency
Document length
```

You don't need to memorize the formula for most backend interviews.

Understand the idea:

> More relevant documents receive higher scores.

---

## 20. Search Ranking

A product search might rank using:

```text
Text relevance
Popularity
Availability
Price
Rating
Business rules
```

The final ranking can combine search relevance with application-specific signals.

---

## 21. Boosting

Some fields may be more important.

Example:

```text
Product name -> high importance
Description -> lower importance
```

Conceptually:

```text
name match      x 3
description match x 1
```

This is field boosting.

---

## 22. Fuzzy Search

Fuzzy search can tolerate small spelling mistakes.

Example:

```text
iphnoe
```

can potentially match:

```text
iphone
```

Useful for:

```text
User typos
Product names
Search boxes
```

Trade-off:

```text
More computation
Potential unwanted matches
```

---

## 23. Prefix Search

Useful for autocomplete.

User types:

```text
spr
```

Results might include:

```text
spring
spring boot
spring security
```

---

## 24. Autocomplete Architecture

```text
User types
   |
   v
Frontend
   |
   v
Search API
   |
   v
Elasticsearch
   |
   v
Suggestions
```

For a good user experience:

```text
Debounce requests
Limit result count
Cache popular prefixes
```

---

## 25. Debouncing

Without debouncing:

```text
s
sp
spr
spri
sprin
spring
```

may generate many API calls.

With debouncing:

```text
Wait briefly
     |
     v
Search "spring"
```

This reduces unnecessary traffic.

---

## 26. Search-as-You-Type

Search engines can support fields specifically designed for incremental typing.

Useful for:

```text
Autocomplete
Search suggestions
Product names
Location search
```

---

## 27. Synonyms

Users may search:

```text
mobile
```

while the catalog uses:

```text
phone
```

Synonym handling can map related terms.

Example:

```text
mobile <-> phone
```

Be careful with synonym rules because they affect indexing and search behavior.

---

## 28. Typo Tolerance

Common techniques:

```text
Fuzzy matching
Spell correction
Synonyms
Autocomplete
Phonetic matching
```

The appropriate technique depends on the product.

---

## 29. Faceted Search

E-commerce search often provides:

```text
Brand
Category
Price
Rating
Availability
```

Example:

```text
Phones
 |
 +--> Apple
 +--> Samsung
 +--> Google
```

These are often implemented using aggregations.

---

## 30. Aggregations

Aggregations calculate summaries over search results.

Examples:

```text
Count by brand
Average price
Price ranges
Rating distribution
```

Example:

```text
Apple   -> 120 products
Samsung -> 180 products
Google  -> 60 products
```

---

## 31. Pagination

Basic pagination:

```text
page=1
size=20
```

For deep pagination, traditional offset-based approaches can become expensive.

---

## 32. Search-After

For large result sets, Elasticsearch supports search-after style pagination.

Conceptually:

```text
Page 1
   |
   v
Last sort values
   |
   v
Page 2
```

This can be more efficient than very deep offsets.

---

## 33. Sorting

Search results can be sorted by:

```text
Relevance
Price
Rating
Date
Popularity
```

Be careful:

```text
Sorting by a field
```

and:

```text
Ranking by relevance
```

are different requirements.

---

## 34. Highlighting

Search results can highlight matching terms.

Example:

```text
Java Backend Developer
^^^^
```

Useful for:

```text
Document search
Article search
Product search
```

---

## 35. Elasticsearch Document

A document is a JSON-like record.

Example:

```json
{
  "id": 101,
  "name": "Java Backend Course",
  "category": "education",
  "price": 999
}
```

Documents are stored in indexes.

---

## 36. Index

An index is a logical collection of documents.

Example:

```text
products
```

contains:

```text
Product 101
Product 102
Product 103
```

An index is roughly comparable to a collection conceptually, but Elasticsearch's architecture and terminology are different from a relational database.

---

## 37. Shards

An Elasticsearch index can be divided into shards.

```text
Index
 |
 +--> Shard 1
 +--> Shard 2
 +--> Shard 3
```

Each shard contains a subset of the index's documents.

Sharding allows data and search work to scale across nodes.

---

## 38. Replicas

A shard can have replica copies.

```text
Primary Shard
     |
     +--> Replica
```

Replicas improve:

```text
Availability
Read capacity
Failure tolerance
```

---

## 39. Cluster

An Elasticsearch cluster contains multiple nodes.

```text
Cluster
 |
 +--> Node 1
 +--> Node 2
 +--> Node 3
```

Indexes and shards are distributed across the cluster.

---

## 40. Primary Shard vs Replica

Conceptually:

```text
Index
 |
 +--> Primary Shard
 |       |
 |       +--> Replica
 |
 +--> Primary Shard
         |
         +--> Replica
```

The exact placement is managed by Elasticsearch.

---

## 41. Scaling Search

Vertical scaling:

```text
Bigger nodes
```

Horizontal scaling:

```text
More nodes
```

Elasticsearch is designed for horizontal distribution through shards and replicas.

---

## 42. Search Fan-Out

A query against a sharded index may involve multiple shards.

Conceptually:

```text
             Search Request
                  |
        +---------+---------+
        |         |         |
        v         v         v
     Shard 1   Shard 2   Shard 3
        |         |         |
        +---------+---------+
                  |
                  v
             Merge Results
```

This introduces network and coordination overhead.

---

## 43. Why Too Many Shards Are Bad

More shards are not always better.

Too many shards can increase:

```text
Memory usage
Metadata overhead
Coordination cost
Recovery time
Management complexity
```

Choose shard counts based on expected data size and workload.

---

## 44. Hot Shards

A shard can receive disproportionately high traffic.

```text
Shard 1 -> 10,000 queries/sec
Shard 2 -> 500 queries/sec
Shard 3 -> 500 queries/sec
```

This creates a hotspot.

Possible causes:

```text
Poor distribution
Hot keys
Uneven data
Popular queries
```

---

## 45. Indexing Pipeline

A common data flow:

```text
MySQL
  |
  v
Change/Event
  |
  v
Indexer
  |
  v
Elasticsearch
```

The search index is updated asynchronously.

---

## 46. Event-Driven Indexing

Example:

```text
Product Updated
      |
      v
Kafka
      |
      v
Search Indexer
      |
      v
Elasticsearch
```

Benefits:

```text
Loose coupling
Retry support
Asynchronous indexing
Scalability
```

---

## 47. Eventual Consistency in Search

Database:

```text
Product price = 999
```

Search index may temporarily contain:

```text
Price = 899
```

until the indexer processes the update.

This is search-index eventual consistency.

---

## 48. Handling Indexing Failure

If indexing fails:

```text
Database updated
      |
      v
Indexer fails
```

Use:

```text
Retry
Dead-letter queue
Reprocessing
Monitoring
Periodic reconciliation
```

The database remains the source of truth.

---

## 49. Full Reindex

Sometimes the index must be rebuilt.

```text
Database
   |
   v
Indexer
   |
   v
New Elasticsearch Index
```

Reasons:

```text
Mapping changes
Analyzer changes
Data corruption
Migration
Improved indexing strategy
```

---

## 50. Zero-Downtime Reindex

A common pattern:

```text
products_v1
products_v2
```

Build the new index:

```text
DB -> products_v2
```

Then switch an alias:

```text
products
   |
   v
products_v2
```

This avoids taking search offline during reindexing.

---

## 51. Index Alias

Instead of applications directly referencing:

```text
products_v1
```

use:

```text
products
```

as an alias.

Then:

```text
products -> products_v1
```

can later become:

```text
products -> products_v2
```

This makes migrations safer.

---

## 52. Search Index as a Derived View

Think of Elasticsearch as:

```text
Source of Truth
       |
       v
Derived Search View
```

If the search index is lost:

```text
Rebuild from source
```

This mindset simplifies architecture.

---

## 53. Search Security

Protect search infrastructure with:

```text
Authentication
Authorization
Network isolation
TLS
Access controls
```

Do not expose an Elasticsearch cluster directly to the public internet without appropriate security controls.

---

## 54. Search Query Security

Don't blindly accept arbitrary query structures from clients.

Validate:

```text
Allowed fields
Allowed filters
Result size
Sort fields
Query complexity
```

This prevents expensive or abusive queries.

---

## 55. Result Size Limits

A client requesting:

```text
size = 1,000,000
```

can create severe load.

Enforce:

```text
Maximum page size
Maximum aggregation complexity
Timeouts
Query limits
```

---

## 56. Search Caching

Cache popular queries:

```text
"iphone 15"
```

Possible layers:

```text
Application cache
Redis
CDN for suitable responses
Elasticsearch caching
```

Be careful with highly dynamic queries.

---

## 57. Search Performance

Important factors:

```text
Query complexity
Number of shards
Shard size
Result size
Aggregations
Filters
Hardware
Cache hit rate
Index design
```

Avoid optimizing only the Elasticsearch query while ignoring the architecture around it.

---

## 58. Search Monitoring

Monitor:

```text
Query latency
p95/p99 latency
Error rate
Indexing rate
Indexing failures
Cluster health
CPU
Memory
Disk
Shard health
Search thread pools
```

---

## 59. Indexing vs Search Performance

A system may have:

```text
Fast search
but slow indexing
```

or:

```text
Fast indexing
but expensive search
```

Balance both workloads.

---

## 60. Bulk Indexing

Instead of sending documents one at a time:

```text
Document 1 -> Elasticsearch
Document 2 -> Elasticsearch
Document 3 -> Elasticsearch
```

use bulk operations:

```text
Batch
 |
 +--> Document 1
 +--> Document 2
 +--> Document 3
```

This reduces request overhead and improves throughput.

---

## 61. Backpressure

If:

```text
Database updates
       |
       v
Indexer
```

produces events faster than Elasticsearch can process them:

```text
Queue grows
```

Use:

```text
Backpressure
Consumer scaling
Batching
Rate control
```

---

## 62. Search Architecture for E-Commerce

```text
                    Client
                      |
                      v
                  Search API
                      |
                      v
               Elasticsearch
                /    |                    v     v      v
            Search Filters Aggregations

Product Service
      |
      v
    MySQL
      |
      v
   Product Events
      |
      v
     Kafka
      |
      v
 Search Indexer
      |
      v
 Elasticsearch
```

MySQL:

```text
Source of truth
```

Elasticsearch:

```text
Search/read model
```

Kafka:

```text
Change propagation
```

---

## 63. Search Architecture for Documents

```text
Upload
  |
  v
Object Storage
  |
  v
Processing Worker
  |
  +--> Extract text
  |
  v
Indexer
  |
  v
Elasticsearch
```

Search can then operate on extracted document text.

---

## 64. Search Architecture for Logs

```text
Applications
     |
     v
Log Collector
     |
     v
Elasticsearch
     |
     v
Kibana
```

Useful for:

```text
Error search
Log filtering
Operational investigation
Dashboards
```

---

## 65. Interview — Why Elasticsearch Instead of MySQL LIKE?

> "For simple search, MySQL can be enough. But when we need full-text search, relevance ranking, fuzzy matching, autocomplete, faceting and large-scale distributed search, Elasticsearch provides data structures and capabilities designed specifically for those workloads."

---

## 66. Interview — What Is an Inverted Index?

> "An inverted index maps terms to the documents containing them. Instead of scanning every document for a word, the search engine can look up the term and quickly find matching documents."

---

## 67. Interview — Is Elasticsearch the Source of Truth?

> "Usually no. I'd keep the primary business data in a transactional database and treat Elasticsearch as a derived search index. If the index is lost or corrupted, it should be possible to rebuild it from the source data."

---

## 68. Interview — How Do You Keep MySQL and Elasticsearch in Sync?

> "I'd publish product changes through an event or change-data-capture pipeline and have an indexing service update Elasticsearch asynchronously. I'd use retries, a dead-letter mechanism and reconciliation jobs because the search index is eventually consistent."

---

## 69. Interview — What Happens if Elasticsearch Is Down?

> "I'd avoid making the search cluster a single point of failure for the core transactional system. Depending on the product, I'd return a degraded search response, use cached popular results, retry transient failures with limits, or temporarily disable search while core operations continue."

---

## 70. Interview — How Would You Implement Autocomplete?

> "I'd use a search index optimized for prefix or search-as-you-type queries. The frontend would debounce requests, the API would limit result size, and I'd cache popular suggestions where useful."

---

## 71. Interview — How Do You Scale Elasticsearch?

> "I'd scale horizontally by adding nodes and distributing indexes across shards, while using replicas for availability and read capacity. I'd monitor shard sizes, hot shards, query latency and resource utilization, and avoid creating excessive numbers of shards."

---

## 72. Practical Scenario — Product Updated in MySQL but Search Shows Old Data

This is likely eventual consistency.

Check:

```text
Product event published?
Indexer running?
Consumer lag?
Indexing failures?
Elasticsearch health?
```

Then:

```text
Retry
Reprocess event
Reconcile index
```

---

## 73. Practical Scenario — Search Became Slow

Check:

```text
p95/p99 latency
Query complexity
Result size
Aggregations
Shard count
Hot shards
CPU
Memory
Disk
Cache hit rate
```

Don't immediately add more nodes without identifying the bottleneck.

---

## 74. Practical Scenario — Search Index Is Corrupted

If MySQL is the source of truth:

```text
Create new index
       |
       v
Reindex from MySQL
       |
       v
Validate
       |
       v
Switch alias
```

This supports zero/minimal-downtime recovery.

---

## 75. Practical Scenario — Search Traffic Suddenly Spikes

Possible protections:

```text
Rate limiting
Query caching
Result-size limits
Autoscaling
Load shedding
Cached popular queries
```

Also investigate whether an expensive query or abusive client is responsible.

---

## 76. Final Checklist

```text
□ Database search vs search engine
□ Elasticsearch
□ Source of truth
□ Search index
□ Inverted index
□ Tokenization
□ Analyzers
□ Stemming
□ Stop words
□ Exact vs full-text search
□ text vs keyword
□ Multi-fields
□ Match query
□ Term query
□ Boolean queries
□ Filters
□ Relevance scoring
□ BM25
□ Boosting
□ Fuzzy search
□ Prefix search
□ Autocomplete
□ Debouncing
□ Search-as-you-type
□ Synonyms
□ Faceted search
□ Aggregations
□ Pagination
□ Search-after
□ Sorting
□ Highlighting
□ Documents
□ Indexes
□ Shards
□ Replicas
□ Clusters
□ Horizontal scaling
□ Search fan-out
□ Hot shards
□ Indexing pipeline
□ Event-driven indexing
□ Eventual consistency
□ Indexing failures
□ Full reindex
□ Zero-downtime reindex
□ Index aliases
□ Search security
□ Query limits
□ Search caching
□ Search monitoring
□ Bulk indexing
□ Backpressure
□ E-commerce architecture
□ Document search
□ Log search
```

---

## 77. One-Minute Interview Answer

### "How would you design product search for an e-commerce application?"

> "I'd keep product data in MySQL as the source of truth and maintain a derived Elasticsearch index for search. Product changes would publish events through Kafka, and a search-indexing service would update Elasticsearch asynchronously. I'd use an inverted index for full-text search, filters and aggregations for facets, and relevance scoring for ranking. For autocomplete I'd use prefix or search-as-you-type queries with frontend debouncing. I'd monitor search latency, indexing failures, consumer lag and shard health, and support full reindexing through a versioned index and alias."

---

## 78. Key Takeaway

> **Treat Elasticsearch as a specialized, scalable search layer rather than your transactional source of truth. Keep authoritative data in the database, build a derived search index, design for eventual consistency, and use indexing, sharding, replicas, caching and controlled queries to scale search safely.**

**File 19 complete.**
