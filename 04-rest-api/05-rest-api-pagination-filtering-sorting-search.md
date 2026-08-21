# REST API — Pagination, Filtering, Sorting and Search

This file covers pagination, filtering, sorting, searching, and efficient collection endpoints in Spring Boot REST APIs.

---

# 1. Why Collection APIs Need Pagination

Suppose an ecommerce API has:

```text
1,000,000 products
```

Returning everything from:

```http
GET /api/products
```

is a bad design.

Problems include:

```text
Large response
High database load
High memory usage
Slow network transfer
Poor client performance
```

Pagination limits how much data is returned per request.

---

# 2. Basic Pagination

A common API design is:

```http
GET /api/products?page=0&size=20
```

Meaning:

```text
page = 0
size = 20
```

The server returns the first 20 records.

The next request:

```http
GET /api/products?page=1&size=20
```

returns the next page.

---

# 3. Spring Data Pageable

Spring Data provides:

```java
Pageable
```

Example:

```java
@GetMapping
public Page<ProductResponse> getProducts(
        Pageable pageable) {

    return productService.getProducts(pageable);
}
```

The client can request:

```text
?page=0&size=20
```

Spring creates the appropriate `Pageable`.

---

# 4. Repository Pagination

Example:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {
}
```

Service:

```java
public Page<ProductResponse> getProducts(
        Pageable pageable) {

    return repository
        .findAll(pageable)
        .map(productMapper::toResponse);
}
```

Spring Data handles the pagination query.

---

# 5. Page vs List

Without pagination:

```java
List<Product> products
```

The application may load every matching row.

With:

```java
Page<Product> products
```

Spring Data can retrieve only the requested page and provide pagination metadata.

---

# 6. Page Metadata

A response can contain:

```text
content
page number
page size
total elements
total pages
first
last
```

Conceptually:

```json
{
  "content": [],
  "page": 0,
  "size": 20,
  "totalElements": 125,
  "totalPages": 7,
  "first": true,
  "last": false
}
```

The exact JSON structure depends on the API design and Spring configuration.

---

# 7. Why Total Count Can Be Expensive

A `Page` often requires both:

```text
SELECT page data
```

and:

```text
SELECT COUNT(...)
```

For very large or complex queries, counting can become expensive.

If the client doesn't need total counts, another approach can be more efficient.

---

# 8. Slice

Spring Data also provides:

```java
Slice<T>
```

A `Slice` can determine whether another page exists without necessarily calculating the complete total count.

Conceptually:

```text
Page
→ content + total count

Slice
→ content + hasNext
```

Use `Slice` when total counts are unnecessary.

---

# 9. Cursor Pagination

Offset pagination:

```text
?page=0&size=20
?page=1&size=20
```

Cursor pagination uses a position from the previous result.

Example:

```http
GET /api/products?limit=20&after=eyJpZCI6MTAw...
```

The cursor represents where the next query should continue.

---

# 10. Offset vs Cursor Pagination

Offset:

```text
page=1000
```

Advantages:

```text
Simple
Easy to understand
Easy for UI page numbers
```

Disadvantages:

```text
Large offsets can become expensive
Results can shift when data changes
```

Cursor:

```text
after=<cursor>
```

Advantages:

```text
Efficient for large datasets
Stable traversal
Good for feeds
```

Disadvantages:

```text
More complex
Harder to jump directly to page 100
```

---

# 11. When to Use Cursor Pagination

Good candidates:

```text
Social feeds
Activity streams
Large ecommerce catalogs
Event logs
Infinite scrolling
High-volume APIs
```

Traditional page numbers are often fine for smaller administrative screens.

---

# 12. Sorting

Clients often need sorted results.

Example:

```http
GET /api/products?sort=price,asc
```

or:

```http
GET /api/products?sort=name,asc
```

Spring Data supports `Sort` through `Pageable`.

---

# 13. Multiple Sort Fields

Example:

```http
GET /api/products?sort=category,asc&sort=price,desc
```

Conceptually:

```text
category ASC
price DESC
```

This can provide deterministic ordering.

---

# 14. Why Deterministic Sorting Matters

Suppose two products have the same:

```text
price = 100
```

If sorting only by price, their order may not be stable.

A secondary key can help:

```text
price ASC
id ASC
```

This is particularly important for pagination.

---

# 15. Filtering

Filtering allows clients to narrow results.

Example:

```http
GET /api/products?category=phones
```

Multiple filters:

```http
GET /api/products
    ?category=phones
    &brand=Apple
    &minPrice=500
    &maxPrice=1500
```

---

# 16. Filtering vs Searching

Filtering:

```text
category = phones
price >= 500
brand = Apple
```

Search:

```text
query = "iphone"
```

Filtering usually matches structured fields.

Search often involves text relevance.

---

# 17. Optional Query Parameters

Example:

```java
@GetMapping
public Page<ProductResponse> searchProducts(
        @RequestParam(required = false)
        String category,

        @RequestParam(required = false)
        BigDecimal minPrice,

        @RequestParam(required = false)
        BigDecimal maxPrice,

        Pageable pageable) {

    return productService.search(
        category,
        minPrice,
        maxPrice,
        pageable
    );
}
```

---

# 18. Avoid Too Many Controller Parameters

If an endpoint has many filters:

```text
category
brand
minPrice
maxPrice
status
rating
createdAfter
createdBefore
```

the controller can become difficult to maintain.

A filter DTO or specification object can provide a cleaner design.

---

# 19. Filter DTO

Example:

```java
public record ProductFilter(
    String category,
    String brand,
    BigDecimal minPrice,
    BigDecimal maxPrice,
    ProductStatus status
) {
}
```

The service can then receive:

```java
ProductFilter filter
```

instead of many individual parameters.

---

# 20. Specification Pattern

Spring Data JPA supports specifications for dynamic filtering.

Conceptually:

```java
Specification<Product> specification =
    Specification.where(categoryEquals(category))
        .and(priceGreaterThan(minPrice))
        .and(priceLessThan(maxPrice));
```

Then:

```java
repository.findAll(
    specification,
    pageable
);
```

This is useful when filters are optional and combinations become complex.

---

# 21. Criteria Queries

JPA Criteria APIs can construct queries dynamically.

However, raw Criteria code can become verbose.

For many applications:

```text
Spring Data Specifications
```

or a query-building library can provide a cleaner abstraction.

---

# 22. Query Method Filtering

For simple cases, Spring Data query methods can be enough.

Example:

```java
List<Product> findByCategory(
    String category
);
```

Another example:

```java
List<Product> findByPriceBetween(
    BigDecimal min,
    BigDecimal max
);
```

---

# 23. Combining Filters

Example:

```java
Page<Product> findByCategoryAndPriceBetween(
    String category,
    BigDecimal min,
    BigDecimal max,
    Pageable pageable
);
```

This works for simple combinations.

For highly dynamic filtering, specifications or custom queries are usually easier to maintain.

---

# 24. Search Endpoint Design

Possible design:

```http
GET /api/products?query=laptop
```

or:

```http
GET /api/products/search?query=laptop
```

Both can work.

Choose a consistent API convention.

---

# 25. Database Search

For simple search:

```sql
WHERE name LIKE '%laptop%'
```

This can be acceptable for small datasets.

For large-scale search, consider appropriate indexing or dedicated search infrastructure.

---

# 26. Search and Indexes

A query such as:

```sql
WHERE sku = ?
```

can be fast when:

```text
sku has an index
```

Without appropriate indexes, the database may need to scan many rows.

Always inspect query plans for important queries.

---

# 27. Pagination + Indexing

Suppose:

```http
GET /products?category=phones&page=0&size=20
```

A suitable index can improve filtering and ordering.

Potentially:

```text
(category, price, id)
```

But indexes should be based on actual query patterns rather than created blindly.

---

# 28. Sorting + Indexing

If an API frequently executes:

```sql
WHERE category = ?
ORDER BY price ASC
```

a suitable composite index may help.

The exact index depends on:

```text
Database
Query shape
Data distribution
Selectivity
Ordering
```

---

# 29. Maximum Page Size

Do not blindly allow:

```http
?size=1000000
```

A safer API may enforce:

```text
default size = 20
maximum size = 100
```

This protects the application from oversized requests.

---

# 30. Spring Pageable Limits

You can configure a maximum page size using Spring MVC configuration.

Conceptually:

```java
@Bean
public PageableHandlerMethodArgumentResolverCustomizer
pageableCustomizer() {

    return resolver ->
        resolver.setMaxPageSize(100);
}
```

The exact configuration can vary by Spring Boot version.

---

# 31. Validate Pagination Parameters

Invalid:

```text
page = -1
size = 0
size = 100000
```

The API should normalize or reject invalid values according to its contract.

---

# 32. Default Pagination

If the client sends:

```http
GET /api/products
```

the API can use:

```text
page = 0
size = 20
```

This prevents accidental unbounded queries.

---

# 33. Sorting Whitelist

Be careful with client-controlled sort fields.

Don't blindly accept arbitrary database field names.

Allowed:

```text
name
price
createdAt
```

Rejected:

```text
internalField
password
unknownColumn
```

A whitelist protects the API contract and prevents unexpected query behavior.

---

# 34. Mapping Sort Fields

The API can expose:

```text
created
```

while internally mapping it to:

```text
createdAt
```

This decouples the public API from database/entity naming.

---

# 35. Filtering Security

Do not allow clients to construct arbitrary SQL.

Bad idea:

```text
?filter=raw SQL
```

Instead use:

```text
Structured parameters
Validated fields
Predefined operators
```

The persistence layer should construct the query safely.

---

# 36. Query Parameter Encoding

Values containing spaces or special characters should be URL encoded.

Example:

```text
query=wireless%20headphones
```

Clients and HTTP libraries generally handle this automatically.

---

# 37. Empty Results

Suppose:

```http
GET /api/products?category=unknown
```

No products match.

A collection endpoint should normally return:

```http
200 OK
```

with an empty collection:

```json
{
  "content": [],
  "totalElements": 0
}
```

Do not return 404 merely because a collection is empty.

---

# 38. Single Resource vs Collection

Important distinction:

```http
GET /products/999
```

If product 999 doesn't exist:

```text
404 Not Found
```

But:

```http
GET /products?category=unknown
```

with no matching products:

```text
200 OK
empty collection
```

---

# 39. Pagination Response DTO

Instead of exposing framework-specific pagination JSON, an API can define its own contract.

Example:

```java
public record PageResponse<T>(
    List<T> content,
    int page,
    int size,
    long totalElements,
    int totalPages
) {
}
```

This gives the API more control over its public contract.

---

# 40. Service Mapping

Example:

```java
public PageResponse<ProductResponse> getProducts(
        Pageable pageable) {

    Page<Product> page =
        repository.findAll(pageable);

    return new PageResponse<>(
        page.map(mapper::toResponse).getContent(),
        page.getNumber(),
        page.getSize(),
        page.getTotalElements(),
        page.getTotalPages()
    );
}
```

---

# 41. Avoid Returning Entities

Prefer:

```text
Entity
  ↓
Mapper
  ↓
DTO
  ↓
API response
```

Instead of:

```text
Entity
  ↓
JSON
```

DTOs help prevent:

```text
Internal fields leaking
Lazy-loading problems
Unstable API contracts
Unwanted relationships
```

---

# 42. Pagination and DTO Mapping

Spring Data makes mapping straightforward:

```java
Page<ProductResponse> response =
    productPage.map(productMapper::toResponse);
```

The pagination metadata remains available.

---

# 43. Large Dataset Considerations

For millions of rows:

```text
Offset pagination
```

may become expensive for high page numbers.

Consider:

```text
Cursor pagination
Keyset pagination
```

for high-volume APIs.

---

# 44. Keyset Pagination

Instead of:

```sql
OFFSET 100000
LIMIT 20
```

use a boundary:

```sql
WHERE id > 100000
ORDER BY id
LIMIT 20
```

This can be much more efficient for sequential traversal.

---

# 45. Keyset Pagination Requirements

Keyset pagination needs a stable ordering.

Example:

```text
createdAt DESC
id DESC
```

The cursor can contain the last seen values:

```text
createdAt
id
```

The next query continues from that position.

---

# 46. Cursor Security

Do not expose sensitive internal state directly in a cursor.

Instead, encode or sign the cursor.

Conceptually:

```text
Internal values
     ↓
Encode/sign
     ↓
Cursor
```

The server decodes and validates it.

---

# 47. Infinite Scroll

A frontend using infinite scrolling can request:

```http
GET /products?limit=20&after=<cursor>
```

Response:

```json
{
  "items": [],
  "nextCursor": "abc123",
  "hasNext": true
}
```

This is often better than page numbers for feeds.

---

# 48. Sorting Stability

Suppose the query sorts by:

```text
createdAt DESC
```

If many records have the same timestamp, add a tie-breaker:

```text
createdAt DESC
id DESC
```

This makes pagination more stable.

---

# 49. Time-Based Pagination

For activity feeds, a cursor might use:

```text
createdAt
id
```

Example:

```text
createdAt < lastCreatedAt
OR
(createdAt = lastCreatedAt AND id < lastId)
```

This provides stable traversal.

---

# 50. API Example — Product Search

Request:

```http
GET /api/products
    ?category=phones
    &minPrice=500
    &maxPrice=1500
    &sort=price,asc
    &page=0
    &size=20
```

Flow:

```text
Controller
    ↓
ProductFilter
    ↓
Service
    ↓
Specification
    ↓
Repository
    ↓
Database
    ↓
Page<Product>
    ↓
DTO mapping
    ↓
Response
```

---

# 51. Ecommerce Product Endpoint

Example controller:

```java
@GetMapping
public PageResponse<ProductResponse> searchProducts(
        ProductFilter filter,
        Pageable pageable) {

    return productService.search(
        filter,
        pageable
    );
}
```

The exact parameter-binding approach can vary depending on how the filter DTO is designed.

---

# 52. Search Performance

For important search APIs monitor:

```text
p50 latency
p95 latency
p99 latency
Database CPU
Rows examined
Rows returned
Cache hit ratio
Query execution time
```

Don't optimize based only on intuition.

---

# 53. N+1 Problem in Paginated APIs

Pagination does not automatically prevent N+1 queries.

Example:

```text
1 query for products
+
20 queries for categories
```

This can still be slow.

Use appropriate:

```text
Fetch strategies
Projections
JOIN FETCH where appropriate
Entity graphs
DTO queries
```

Be careful with collection fetch joins and pagination because they can create incorrect or inefficient results.

---

# 54. Count Query Performance

For expensive paginated queries:

```text
Data query
+
Count query
```

can become a bottleneck.

If the UI only needs:

```text
hasNext
```

consider `Slice`.

---

# 55. API Caching and Pagination

Cache keys must include all inputs that affect the result.

Example:

```text
products:phones:500:1500:price-asc:0:20
```

If page, filters, or sort order change, the cache key must change too.

---

# 56. Pagination and Consistency

Data can change between requests.

Example:

```text
Request page 0
Product inserted
Request page 1
```

The client may see:

```text
Duplicate item
Missing item
```

Cursor/keyset pagination can provide more stable traversal for changing datasets.

---

# 57. Filtering Null Values

Be explicit about optional filters.

Example:

```text
category = null
minPrice = null
maxPrice = null
```

should mean:

```text
Do not apply those filters
```

Avoid generating unnecessary conditions.

---

# 58. Range Filtering

Typical ecommerce filters:

```text
minPrice
maxPrice
minRating
maxRating
createdAfter
createdBefore
```

Validate ranges:

```text
minPrice <= maxPrice
```

If the range is invalid, return a client error.

---

# 59. Search Result Limits

Search APIs should also enforce limits.

Example:

```text
default = 20
maximum = 100
```

This prevents clients from accidentally requesting huge result sets.

---

# 60. Interview: Why Use Pagination?

> Pagination prevents an API from loading and returning an unnecessarily large dataset in one request. It reduces database load, memory usage, network payloads and response latency.

---

# 61. Interview: Page vs Slice?

> `Page` provides pagination metadata including the total number of matching records and pages, while `Slice` focuses on whether another slice exists and can avoid the cost of calculating a full count. I use `Page` when the UI needs totals and `Slice` when it doesn't.

---

# 62. Interview: Offset vs Cursor Pagination?

> Offset pagination is simple and works well for traditional page-based UIs, but large offsets can become expensive and results can shift when data changes. Cursor or keyset pagination is more suitable for large datasets, feeds and infinite scrolling because traversal is based on a stable position.

---

# 63. Interview: How Do You Prevent Huge Page Requests?

> I define a default page size and enforce a maximum, such as 100 records. This prevents clients from accidentally or intentionally requesting very large result sets.

---

# 64. Interview: How Do You Handle Sorting?

> I expose a controlled list of sortable fields and map them to internal fields when necessary. I also use a deterministic secondary sort key such as ID so pagination remains stable.

---

# 65. Interview: How Do You Implement Dynamic Filtering?

> For simple filters I can use Spring Data query methods. When filters become dynamic and combinations grow, I prefer Specifications or carefully designed custom queries. I validate filter values and avoid allowing arbitrary SQL or database fields from the client.

---

# 66. Interview: Why Not Return All Products?

> Returning all products does not scale. It increases database work, memory usage, response size and network latency. I would use pagination and appropriate database indexes, and for very large datasets I would consider keyset or cursor pagination.

---

# 67. Interview: Why Is Keyset Pagination Faster?

> Keyset pagination uses a known boundary such as the last ID instead of asking the database to skip a large number of rows with a high OFFSET. With an appropriate index, the database can continue from the boundary efficiently.

---

# 68. Interview Scenario — Ecommerce Search

Question:

Design:

```http
GET /api/products
```

with:

```text
category
brand
price range
sorting
pagination
```

Answer:

> I would use query parameters for the filters and Spring Data `Pageable` for pagination and sorting. For dynamic combinations I would use a filter DTO with Specifications or a custom query. I would enforce maximum page sizes, validate ranges, add indexes based on real query patterns, and return DTOs rather than entities.

---

# 69. Interview Scenario — Million-Row Table

Question:

There are 10 million products. Would you use `page=500000`?

Answer:

> I would be cautious with large offsets because the database may need to scan or skip many rows. For high-volume traversal I would consider keyset or cursor pagination using a stable indexed ordering such as `createdAt` plus `id`.

---

# 70. Interview Scenario — Empty Search

Question:

A product search returns zero products. Should it return 404?

Answer:

> No. For a collection endpoint, an empty result is normally a successful response with an empty collection. A 404 is more appropriate when a specific resource requested by ID does not exist.

---

# 71. Final Architecture

```text
                  HTTP REQUEST
                       ↓
                 Controller
                       ↓
              Filter + Pageable
                       ↓
                    Service
                       ↓
          +------------+------------+
          |                         |
     Simple filters          Dynamic filters
          |                         |
    Repository method        Specification
          |                         |
          +------------+------------+
                       ↓
                    Database
                       ↓
                  Page / Slice
                       ↓
                    DTO Map
                       ↓
                 API Response
```

---

# 72. Final Checklist

```text
□ Use pagination for large collections
□ Define default page size
□ Enforce maximum page size
□ Validate page and filter values
□ Use Page when totals are needed
□ Use Slice when totals are unnecessary
□ Consider cursor/keyset pagination for large datasets
□ Whitelist sortable fields
□ Use deterministic sorting
□ Validate filter ranges
□ Avoid arbitrary SQL from clients
□ Add indexes based on query patterns
□ Return DTOs
□ Watch for N+1 queries
□ Consider count-query cost
□ Include all query dimensions in cache keys
□ Test empty results
□ Test large datasets
□ Monitor query performance
```

---

# Final Mental Model

```text
Client
  |
  | filters + sort + pagination
  ↓
Controller
  ↓
Service
  ↓
Query strategy
  |
  +-- simple → Repository method
  |
  +-- dynamic → Specification / custom query
  |
  ↓
Database
  ↓
Page / Slice
  ↓
DTO
  ↓
Response
```

> **A scalable collection API should never assume the dataset is small. Use pagination, validate and limit client-controlled parameters, keep sorting deterministic, choose offset or cursor pagination based on the workload, and make the database query efficient with the right indexes.**
