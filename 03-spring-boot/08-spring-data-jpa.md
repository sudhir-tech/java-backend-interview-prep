# Spring Boot — Spring Data JPA

Spring Data JPA is one of the most important parts of a Spring Boot backend application.

A typical data-access flow is:

```text
REST Controller
      ↓
Service
      ↓
Spring Data Repository
      ↓
JPA / Hibernate
      ↓
JDBC
      ↓
Database
```

The important concepts are:

```text
JPA
Hibernate
Entity
Repository
EntityManager
Persistence Context
CRUD
Derived Queries
JPQL
Native SQL
Relationships
Fetch Types
Cascade
Transactions
Pagination
Sorting
Projections
Specifications
Optimistic Locking
N+1 Queries
```

---

# 1. What Is JPA?

JPA stands for:

```text
Jakarta Persistence API
```

It is a specification for object-relational mapping and persistence in Java.

JPA defines concepts such as:

```text
@Entity
@Id
@OneToMany
@ManyToOne
EntityManager
JPQL
```

JPA itself is a specification.

It does not implement the persistence behavior.

---

# 2. What Is Hibernate?

Hibernate is a popular JPA implementation.

Conceptually:

```text
JPA
 ↓
Specification

Hibernate
 ↓
Implementation
```

Spring Boot commonly uses Hibernate as the JPA provider.

---

# 3. JPA vs Hibernate

Interview answer:

> JPA is a specification that defines standard persistence APIs and annotations, while Hibernate is an implementation of that specification. In a Spring Boot application, I can use JPA APIs while Hibernate performs the underlying ORM work.

---

# 4. What Is ORM?

ORM means:

```text
Object Relational Mapping
```

It maps:

```text
Java Object
      ↕
Database Row
```

Example:

```java
@Entity
public class Product {

    @Id
    private Long id;

    private String name;

    private BigDecimal price;
}
```

Can map conceptually to:

```text
products
--------------------------------
id | name | price
--------------------------------
1  | Laptop | 75000
```

---

# 5. @Entity

Marks a Java class as a JPA entity.

```java
@Entity
public class Product {

    @Id
    private Long id;

    private String name;
}
```

The entity is mapped to a database table.

---

# 6. @Table

Specify the table name:

```java
@Entity
@Table(name = "products")
public class Product {

}
```

Without `@Table`, the provider can derive a table name based on its naming strategy.

---

# 7. @Id

Every JPA entity needs an identifier.

```java
@Id
private Long id;
```

The identifier uniquely identifies an entity instance.

---

# 8. @GeneratedValue

Defines ID generation.

```java
@Id
@GeneratedValue(
    strategy = GenerationType.IDENTITY
)
private Long id;
```

Common strategies include:

```text
IDENTITY
SEQUENCE
TABLE
AUTO
```

The best strategy depends on the database and application requirements.

---

# 9. GenerationType.IDENTITY

Example:

```java
@Id
@GeneratedValue(
    strategy = GenerationType.IDENTITY
)
private Long id;
```

The database typically generates the ID using an identity/auto-increment mechanism.

Common with MySQL.

---

# 10. GenerationType.SEQUENCE

Example:

```java
@Id
@GeneratedValue(
    strategy = GenerationType.SEQUENCE
)
private Long id;
```

Uses a database sequence where supported.

Common with databases such as PostgreSQL and Oracle.

---

# 11. Entity Example

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(
        strategy = GenerationType.IDENTITY
    )
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private BigDecimal price;

    // constructors
    // getters
    // setters
}
```

---

# 12. @Column

Customize column mapping.

```java
@Column(
    name = "product_name",
    nullable = false,
    length = 100
)
private String name;
```

Common attributes:

```text
name
nullable
length
unique
precision
scale
insertable
updatable
```

---

# 13. Nullable

Example:

```java
@Column(nullable = false)
private String name;
```

This can contribute to database schema generation and metadata.

It should not replace API validation.

You may also use:

```java
@NotBlank
```

for request validation.

---

# 14. Unique

Example:

```java
@Column(unique = true)
private String sku;
```

For production database design, explicit database constraints and indexes are often preferable to relying only on generated schema.

---

# 15. @Enumerated

For enums:

```java
@Enumerated(EnumType.STRING)
private OrderStatus status;
```

Example:

```java
public enum OrderStatus {

    CREATED,
    PAID,
    SHIPPED,
    CANCELLED
}
```

Prefer:

```text
EnumType.STRING
```

over ordinal storage in most business applications.

---

# 16. Why Avoid EnumType.ORDINAL?

Ordinal stores:

```text
CREATED   → 0
PAID      → 1
SHIPPED   → 2
```

If you reorder the enum:

```text
CREATED
SHIPPED
PAID
```

the stored numeric meanings can become incorrect.

String storage is more stable.

---

# 17. @Transient

A JPA `@Transient` field is not persisted.

```java
@Transient
private BigDecimal calculatedDiscount;
```

It exists in the Java object but is not mapped to a database column.

Do not confuse:

```text
jakarta.persistence.Transient
```

with Java's:

```text
transient
```

They solve different problems.

---

# 18. @Version

Used for optimistic locking.

```java
@Version
private Long version;
```

When an entity is updated, JPA checks the version.

Conceptually:

```text
Read version 5
      ↓
Update with version 5
      ↓
Version becomes 6
```

If another transaction already changed it:

```text
Database version = 6
Application expects = 5
```

the update can fail with an optimistic locking exception.

---

# 19. Spring Data JPA

Spring Data JPA provides repository abstractions over JPA.

Instead of implementing CRUD manually:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {

}
```

you immediately get methods such as:

```text
save()
findById()
findAll()
deleteById()
existsById()
count()
```

---

# 20. Repository Hierarchy

Conceptually:

```text
Repository
   ↓
CrudRepository
   ↓
PagingAndSortingRepository
   ↓
JpaRepository
```

The exact inheritance structure can vary across Spring Data versions, but `JpaRepository` provides a rich JPA-oriented repository API.

---

# 21. JpaRepository

Example:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {

}
```

This provides CRUD and pagination/sorting functionality.

---

# 22. save()

```java
Product product =
    repository.save(product);
```

`save()` can be used for both:

```text
Persisting new entities
Merging existing detached entities
```

The exact behavior depends on whether Spring Data considers the entity new.

Do not think of `save()` as simply:

```text
always INSERT
```

or:

```text
always UPDATE
```

---

# 23. findById()

```java
Optional<Product> product =
    repository.findById(id);
```

Recommended handling:

```java
Product product =
    repository.findById(id)
        .orElseThrow(
            () -> new ProductNotFoundException(id)
        );
```

---

# 24. findAll()

```java
List<Product> products =
    repository.findAll();
```

Do not use this blindly for very large tables.

For large datasets, prefer:

```text
Pagination
Filtering
Streaming where appropriate
Batch processing
```

---

# 25. deleteById()

```java
repository.deleteById(id);
```

The repository removes the entity with that identifier.

Business rules should normally be handled by the service before deletion.

---

# 26. existsById()

```java
boolean exists =
    repository.existsById(id);
```

Useful for checks such as:

```text
Duplicate resource
Referenced resource
Business validation
```

But avoid unnecessary existence queries when the actual operation already provides the required constraint/error handling.

---

# 27. count()

```java
long count =
    repository.count();
```

Returns the number of entities represented by the repository.

For very large datasets, understand the cost of count queries.

---

# 28. Derived Query Methods

Spring Data can create queries from method names.

Example:

```java
List<Product>
findByCategory(String category);
```

More complex:

```java
List<Product>
findByCategoryAndPriceGreaterThan(
    String category,
    BigDecimal price
);
```

---

# 29. Common Derived Query Keywords

Examples:

```text
findBy
findAllBy
existsBy
countBy
deleteBy
removeBy

And
Or
Between
LessThan
LessThanEqual
GreaterThan
GreaterThanEqual
Like
Containing
StartingWith
EndingWith
In
NotIn
IsNull
IsNotNull
OrderBy
```

---

# 30. Example Derived Queries

```java
List<Product>
findByPriceGreaterThan(
    BigDecimal price
);
```

```java
List<Product>
findByNameContainingIgnoreCase(
    String keyword
);
```

```java
List<Product>
findByCategoryAndPriceBetween(
    String category,
    BigDecimal min,
    BigDecimal max
);
```

---

# 31. When Derived Queries Become Too Long

Bad:

```java
findByCategoryAndBrandAndPriceGreaterThanAndPriceLessThanAndStatusAndCreatedAtBetween(...)
```

Very long method names become difficult to understand and maintain.

Prefer:

```text
@Query
Specification
QueryDSL
Custom repository
```

depending on project requirements.

---

# 32. @Query

Define JPQL explicitly:

```java
@Query("""
    SELECT p
    FROM Product p
    WHERE p.category = :category
""")
List<Product> findByCategory(
    @Param("category")
    String category
);
```

This is JPQL, not SQL.

---

# 33. JPQL

JPQL operates on:

```text
Entities
```

and:

```text
Entity fields
```

not database table/column names.

Example:

```java
SELECT p
FROM Product p
WHERE p.price > :price
```

Here:

```text
Product = entity
price   = entity field
```

---

# 34. JPQL vs SQL

JPQL:

```sql
SELECT p
FROM Product p
WHERE p.price > :price
```

SQL:

```sql
SELECT *
FROM products
WHERE price > ?
```

JPQL is object-oriented.

SQL is database-oriented.

---

# 35. @Param

Bind named parameters:

```java
@Query("""
    SELECT p
    FROM Product p
    WHERE p.name = :name
""")
List<Product> findByName(
    @Param("name")
    String name
);
```

---

# 36. Positional Parameters

JPQL can also use positional parameters:

```java
@Query("""
    SELECT p
    FROM Product p
    WHERE p.price > ?1
""")
List<Product> findExpensiveProducts(
    BigDecimal price
);
```

Named parameters are generally easier to read.

---

# 37. Native Query

You can execute database SQL:

```java
@Query(
    value = """
        SELECT *
        FROM products
        WHERE price > :price
    """,
    nativeQuery = true
)
List<Product> findExpensiveProducts(
    @Param("price")
    BigDecimal price
);
```

Use native SQL when database-specific functionality or a query that is awkward in JPQL is genuinely needed.

---

# 38. JPQL vs Native SQL

Prefer JPQL when:

```text
Query is entity-oriented
Database portability matters
JPA relationships are useful
```

Use native SQL when:

```text
Database-specific features are needed
Complex SQL is easier directly
Performance tuning requires database-specific SQL
Existing database queries must be reused
```

Do not default to native SQL for every query.

---

# 39. Update Query

Example:

```java
@Modifying
@Query("""
    UPDATE Product p
    SET p.price = :price
    WHERE p.id = :id
""")
int updatePrice(
    @Param("id") Long id,
    @Param("price") BigDecimal price
);
```

---

# 40. Why @Modifying?

`@Query` normally represents a select query.

For:

```text
UPDATE
DELETE
```

use:

```java
@Modifying
```

with the appropriate transaction configuration.

---

# 41. Transaction for Modifying Query

Example:

```java
@Transactional
@Modifying
@Query("""
    UPDATE Product p
    SET p.price = :price
    WHERE p.id = :id
""")
int updatePrice(
    @Param("id") Long id,
    @Param("price") BigDecimal price
);
```

A common design is to put transaction boundaries at the service layer.

---

# 42. Persistence Context

The JPA persistence context is a managed set of entity instances.

It acts like a first-level cache and tracks entity state changes.

Conceptually:

```text
Persistence Context
        ↓
Managed Entities
```

Within a persistence context, JPA can track changes automatically.

---

# 43. Entity States

Common JPA entity states:

```text
Transient
Managed
Detached
Removed
```

---

# 44. Transient

A newly created object not yet associated with a persistence context:

```java
Product product =
    new Product();
```

It is:

```text
Transient
```

---

# 45. Managed

Once associated with the persistence context:

```text
Managed
```

JPA tracks changes.

Example:

```java
Product product =
    repository.findById(id)
        .orElseThrow();

product.setPrice(
    BigDecimal.valueOf(70000)
);
```

Inside a transaction, the entity may be managed.

---

# 46. Dirty Checking

Hibernate can detect changes to managed entities.

Example:

```java
@Transactional
public void updatePrice(
        Long id,
        BigDecimal price) {

    Product product =
        repository.findById(id)
            .orElseThrow();

    product.setPrice(price);
}
```

You may not need:

```java
repository.save(product);
```

for an already managed entity.

At transaction flush/commit, dirty checking can generate:

```sql
UPDATE products
SET price = ?
WHERE id = ?
```

---

# 47. Important save() Interview Point

This code:

```java
Product product =
    repository.findById(id)
        .orElseThrow();

product.setPrice(price);
```

inside a transactional method can be enough for an update because:

```text
find
↓
managed entity
↓
modify
↓
dirty checking
↓
flush
```

`save()` may still be used for consistency/readability depending on the repository pattern, but it is not always required for managed updates.

---

# 48. Detached Entity

An entity becomes detached when it is no longer associated with the active persistence context.

Examples can occur when:

```text
Persistence context closes
Entity is explicitly detached
Entity crosses transaction boundaries without remaining managed
```

Changes to a detached object are not automatically tracked.

---

# 49. Removed Entity

An entity marked for deletion:

```text
Removed
```

can be deleted from the database during flush.

Example:

```java
repository.delete(product);
```

---

# 50. Flush

Flush synchronizes pending persistence-context changes with the database.

Conceptually:

```text
Managed entity changes
       ↓
Persistence context
       ↓
flush
       ↓
SQL statements
```

Flush does not necessarily mean transaction commit.

---

# 51. Flush vs Commit

Important distinction:

```text
Flush
→ Synchronize changes with database

Commit
→ Complete transaction
```

A flush can happen before commit.

A transaction can still roll back after a flush.

---

# 52. First-Level Cache

The persistence context acts as the first-level cache.

Within the same persistence context:

```text
find Product 101
      ↓
Database query
      ↓
Product object
      ↓
find Product 101 again
      ↓
Managed instance may be reused
```

This is associated with the EntityManager/persistence context.

---

# 53. Second-Level Cache

Hibernate can also support a second-level cache.

Conceptually:

```text
Application
    ↓
Persistence Context
    ↓
Second-Level Cache
    ↓
Database
```

The second-level cache is optional and requires explicit configuration.

Common providers include:

```text
Ehcache
Caffeine
Redis-based solutions through appropriate integrations
```

The exact caching strategy depends on the application.

---

# 54. EntityManager

JPA provides:

```java
EntityManager
```

It manages:

```text
Persistence context
Entity lifecycle
Queries
Flush
Persist
Merge
Remove
```

Spring Data JPA hides much of this complexity through repositories.

---

# 55. persist()

Conceptually:

```java
entityManager.persist(product);
```

makes a new entity managed.

Used primarily for new/transient entities.

---

# 56. merge()

Conceptually:

```java
Product managed =
    entityManager.merge(detachedProduct);
```

`merge()` copies state from a detached entity into a managed instance.

Important:

> `merge()` does not simply turn the original detached object into the managed instance.

The returned object is the managed instance.

---

# 57. remove()

```java
entityManager.remove(product);
```

marks a managed entity for deletion.

---

# 58. clear()

```java
entityManager.clear();
```

detaches all managed entities from the persistence context.

Useful in some batch-processing scenarios.

---

# 59. detach()

```java
entityManager.detach(product);
```

removes one entity from the persistence context.

---

# 60. Relationships

JPA supports:

```text
@OneToOne
@OneToMany
@ManyToOne
@ManyToMany
```

---

# 61. @ManyToOne

Example:

```java
@Entity
public class Product {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
}
```

Conceptually:

```text
Many Products
      ↓
One Category
```

This is one of the most common relationships.

---

# 62. @OneToMany

Category:

```java
@OneToMany(
    mappedBy = "category"
)
private List<Product> products;
```

Conceptually:

```text
Category
   ↓
Many Products
```

---

# 63. Owning Side

In a bidirectional relationship, one side owns the relationship.

Example:

```java
@ManyToOne
@JoinColumn(name = "category_id")
private Category category;
```

The `Product` side owns the relationship because it controls the foreign-key mapping.

The inverse side:

```java
@OneToMany(mappedBy = "category")
```

uses:

```text
mappedBy
```

---

# 64. mappedBy

Example:

```java
@OneToMany(
    mappedBy = "category"
)
private List<Product> products;
```

`mappedBy` points to the field on the owning entity:

```java
Product.category
```

It prevents the inverse side from creating a separate relationship mapping.

---

# 65. Bidirectional Relationship

Example:

```java
@Entity
public class Category {

    @OneToMany(
        mappedBy = "category"
    )
    private List<Product> products;
}
```

```java
@Entity
public class Product {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
}
```

---

# 66. Keep Both Sides in Sync

If you maintain a bidirectional relationship:

```java
public void addProduct(Product product) {

    products.add(product);
    product.setCategory(this);
}
```

This keeps the Java object graph consistent.

---

# 67. @OneToOne

Example:

```java
@OneToOne
@JoinColumn(name = "address_id")
private Address address;
```

Conceptually:

```text
User
 ↓
One Address
```

Use one-to-one only when the domain genuinely has that cardinality.

---

# 68. @ManyToMany

Example:

```java
@ManyToMany
@JoinTable(
    name = "product_tag",
    joinColumns =
        @JoinColumn(name = "product_id"),
    inverseJoinColumns =
        @JoinColumn(name = "tag_id")
)
private Set<Tag> tags;
```

Database:

```text
product
tag
product_tag
```

Many-to-many relationships can become complex as the domain evolves.

If the join table has its own business data, model it as an explicit entity instead.

---

# 69. Join Entity

Suppose:

```text
Order
Product
```

have:

```text
quantity
priceAtPurchase
```

on the relationship.

Instead of:

```text
@ManyToMany
```

create:

```text
OrderItem
```

Then:

```text
Order
 ↓
OrderItem
 ↓
Product
```

This is usually a better domain model.

---

# 70. Cascade

Cascade determines which operations propagate from one entity to associated entities.

Examples:

```text
PERSIST
MERGE
REMOVE
REFRESH
DETACH
ALL
```

Example:

```java
@OneToMany(
    mappedBy = "order",
    cascade = CascadeType.ALL
)
private List<OrderItem> items;
```

---

# 71. CascadeType.ALL

Means all supported cascade operations are propagated.

```java
cascade = CascadeType.ALL
```

Use carefully.

For example, cascading `REMOVE` from a parent to shared entities can accidentally delete data you intended to keep.

---

# 72. Cascade vs Fetch

Do not confuse:

```text
Cascade
```

with:

```text
Fetch
```

Cascade controls:

```text
Entity operations
```

Fetch controls:

```text
When associated data is loaded
```

---

# 73. FetchType.LAZY

Lazy loading means the association is loaded when accessed, rather than immediately with the initial entity query.

Example:

```java
@ManyToOne(fetch = FetchType.LAZY)
private Category category;
```

This is often preferable for relationships.

---

# 74. FetchType.EAGER

Eager loading means the association is expected to be available immediately.

Example:

```java
@ManyToOne(fetch = FetchType.EAGER)
private Category category;
```

Do not assume EAGER always means one SQL JOIN. The provider can use different loading strategies.

Avoid unnecessary eager relationships because they can increase query complexity and memory usage.

---

# 75. Default Fetch Types

Common JPA defaults:

```text
@ManyToOne → EAGER
@OneToOne  → EAGER

@OneToMany → LAZY
@ManyToMany → LAZY
```

A practical recommendation is to explicitly choose fetch behavior for important relationships rather than relying blindly on defaults.

---

# 76. LazyInitializationException

A common problem:

```text
Transaction/session ends
        ↓
Lazy relationship accessed
        ↓
Persistence context unavailable
        ↓
LazyInitializationException
```

Do not solve this automatically by making every relationship:

```text
EAGER
```

Instead, design query boundaries properly.

---

# 77. Solving Lazy Loading Problems

Common approaches:

```text
Fetch join
EntityGraph
DTO projection
Transaction boundary
Explicit query
```

Choose based on the use case.

---

# 78. N+1 Query Problem

Suppose:

```java
List<Order> orders =
    repository.findAll();
```

Then code accesses:

```java
order.getCustomer()
```

for every order.

Potential result:

```text
1 query for orders
+
N queries for customers
=
N+1 queries
```

This can seriously hurt performance.

---

# 79. JOIN FETCH

JPQL can fetch associated data:

```java
@Query("""
    SELECT o
    FROM Order o
    JOIN FETCH o.customer
""")
List<Order> findOrdersWithCustomers();
```

This can reduce unnecessary additional queries.

Use it intentionally because fetch joins can also produce duplicate rows with collection relationships.

---

# 80. @EntityGraph

Example:

```java
@EntityGraph(
    attributePaths = "customer"
)
List<Order> findAll();
```

This can be a clean way to specify fetch requirements for a repository query.

---

# 81. DTO Projection

Instead of loading complete entities:

```java
@Query("""
    SELECT new com.example.dto.ProductSummary(
        p.id,
        p.name,
        p.price
    )
    FROM Product p
""")
List<ProductSummary> findProductSummaries();
```

This can reduce unnecessary data loading.

---

# 82. Interface Projection

Spring Data can also use interface projections.

Example:

```java
public interface ProductSummary {

    Long getId();

    String getName();

    BigDecimal getPrice();
}
```

Repository:

```java
List<ProductSummary>
findByCategory(String category);
```

This can be useful for read-only API views.

---

# 83. Closed Projection

A projection that maps directly to known entity properties:

```java
public interface ProductSummary {

    String getName();

    BigDecimal getPrice();
}
```

This can allow Spring Data to optimize the selected data depending on the query/provider.

---

# 84. Dynamic Projection

A repository can support different projection types:

```java
<T> List<T> findByCategory(
    String category,
    Class<T> type
);
```

Then:

```java
repository.findByCategory(
    "electronics",
    ProductSummary.class
);
```

This is an advanced Spring Data feature.

---

# 85. Pagination

Repository:

```java
Page<Product> findByCategory(
    String category,
    Pageable pageable
);
```

Service:

```java
Page<Product> products =
    repository.findByCategory(
        category,
        PageRequest.of(0, 20)
    );
```

---

# 86. PageRequest

Example:

```java
Pageable pageable =
    PageRequest.of(
        0,
        20,
        Sort.by(
            Sort.Direction.DESC,
            "price"
        )
    );
```

Meaning:

```text
Page 0
20 records
Sort by price descending
```

---

# 87. Sorting

Repository:

```java
List<Product> findAll(
    Sort sort
);
```

Usage:

```java
Sort sort =
    Sort.by(
        Sort.Direction.ASC,
        "name"
    );

repository.findAll(sort);
```

---

# 88. Pagination + Sorting

```java
Pageable pageable =
    PageRequest.of(
        0,
        20,
        Sort.by("name").ascending()
    );

Page<Product> page =
    repository.findAll(pageable);
```

This is common in production APIs.

---

# 89. Slice vs Page

`Page<T>` provides:

```text
Content
Page number
Page size
Total elements
Total pages
```

`Slice<T>` provides:

```text
Content
Page position
Whether another slice exists
```

`Page` usually requires a count query to determine totals.

If total counts are not needed, `Slice` can sometimes be more efficient.

---

# 90. Specifications

Spring Data JPA Specifications can build dynamic queries.

Useful when filters are optional.

Example:

```text
category?
brand?
minPrice?
maxPrice?
status?
```

Instead of creating many repository methods:

```text
findByCategory...
findByBrand...
findByCategoryAndBrand...
findByCategoryAndBrandAndPrice...
```

Specifications can compose predicates dynamically.

---

# 91. Specification Example

Conceptually:

```java
Specification<Product> spec =
    (root, query, cb) ->
        cb.equal(
            root.get("category"),
            category
        );
```

Then:

```java
repository.findAll(
    spec,
    pageable
);
```

This is especially useful for search/filter APIs.

---

# 92. JpaSpecificationExecutor

Repository:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long>,
                JpaSpecificationExecutor<Product> {

}
```

Then:

```java
Page<Product> findAll(
    Specification<Product> specification,
    Pageable pageable
);
```

---

# 93. Criteria API

JPA provides the Criteria API for programmatic query construction.

It is useful for dynamic queries but can become verbose.

For many applications:

```text
Derived queries
JPQL
Specifications
```

are easier to maintain.

---

# 94. Query by Example

Spring Data also supports Query by Example.

Conceptually:

```java
Product probe =
    new Product();

probe.setCategory(
    "electronics"
);
```

Then construct an example and query.

Useful for simple dynamic matching, but not ideal for every complex search requirement.

---

# 95. Transactions

Database operations should be grouped into appropriate transaction boundaries.

Example:

```java
@Transactional
public void placeOrder(
        OrderRequest request) {

    saveOrder();
    updateInventory();
    savePayment();
}
```

If a transaction fails according to rollback rules:

```text
Changes can be rolled back
```

---

# 96. Transaction Boundary

Prefer transaction boundaries at the service layer:

```text
Controller
    ↓
@Service
@Transactional
    ↓
Repository
```

The service knows the complete business operation.

---

# 97. Read-Only Transaction

Example:

```java
@Transactional(readOnly = true)
public ProductResponse getProduct(
        Long id) {

    ...
}
```

This can communicate intent and may allow provider/database optimizations.

Do not treat it as an absolute security mechanism preventing all writes.

---

# 98. Transaction Propagation

Common propagation modes:

```text
REQUIRED
REQUIRES_NEW
SUPPORTS
MANDATORY
NOT_SUPPORTED
NEVER
NESTED
```

Most everyday service methods use:

```text
REQUIRED
```

which joins an existing transaction or creates one if necessary.

---

# 99. REQUIRES_NEW

```java
@Transactional(
    propagation =
        Propagation.REQUIRES_NEW
)
```

Suspends the existing transaction and starts a new one when supported.

Useful in specific cases such as independent audit operations.

Use carefully because it changes transaction semantics and can affect connection usage.

---

# 100. Isolation Levels

Common isolation levels:

```text
READ_UNCOMMITTED
READ_COMMITTED
REPEATABLE_READ
SERIALIZABLE
```

The actual behavior depends on the database.

Isolation controls how concurrent transactions can observe each other's changes.

---

# 101. Dirty Read

A dirty read occurs when one transaction reads data written by another transaction that has not committed.

Conceptually:

```text
Transaction A
  writes 100
       ↓
Transaction B
  reads 100
       ↓
Transaction A rolls back
```

Transaction B read data that never committed.

---

# 102. Non-Repeatable Read

A transaction reads the same row twice and gets different committed values because another transaction updated it between reads.

---

# 103. Phantom Read

A transaction repeats a query and sees additional or missing rows because another transaction inserted/deleted matching rows.

---

# 104. Optimistic Locking

Use:

```java
@Version
private Long version;
```

Conceptually:

```text
User A reads version 5
User B reads version 5

User A updates
version → 6

User B tries update with version 5
       ↓
Conflict
```

This prevents silent overwrites.

---

# 105. Pessimistic Locking

JPA supports lock modes such as:

```text
PESSIMISTIC_READ
PESSIMISTIC_WRITE
```

Example:

```java
@Lock(
    LockModeType.PESSIMISTIC_WRITE
)
Optional<Product> findById(
    Long id
);
```

The database can lock the relevant row according to its locking semantics.

Use pessimistic locking carefully because it can reduce concurrency and cause lock contention/deadlocks.

---

# 106. Optimistic vs Pessimistic Locking

### Optimistic

```text
Assume conflicts are uncommon
Check version during update
```

### Pessimistic

```text
Lock data during the transaction
Prevent conflicting operations
```

Use optimistic locking for many ordinary business updates.

Use pessimistic locking when the business operation genuinely requires stronger serialization and the database can support it appropriately.

---

# 107. Bulk Updates

Example:

```java
@Modifying
@Query("""
    UPDATE Product p
    SET p.active = false
    WHERE p.category = :category
""")
int deactivateByCategory(
    @Param("category")
    String category
);
```

Bulk operations operate directly against the database and can bypass normal per-entity dirty checking behavior.

This creates an important issue:

```text
Persistence context may contain stale entities.
```

---

# 108. clearAutomatically

For bulk modifying queries, you may see:

```java
@Modifying(
    clearAutomatically = true
)
```

This can clear the persistence context after the query.

Use it intentionally because clearing detaches managed entities.

---

# 109. Bulk Update vs Entity Update

Entity update:

```text
Load entity
↓
Modify object
↓
Dirty checking
↓
SQL
```

Bulk update:

```text
Execute UPDATE directly
↓
Database
```

Bulk update can be faster for large batches but requires more care around persistence-context consistency.

---

# 110. Database Indexes

JPA can define indexes:

```java
@Table(
    name = "products",
    indexes = {
        @Index(
            name = "idx_product_sku",
            columnList = "sku"
        )
    }
)
```

Indexes can improve read performance.

But too many indexes can:

```text
Increase storage
Slow writes
Increase maintenance cost
```

Index based on actual query patterns.

---

# 111. Unique Constraints

Example:

```java
@Table(
    name = "products",
    uniqueConstraints = {
        @UniqueConstraint(
            name = "uk_product_sku",
            columnNames = "sku"
        )
    }
)
```

Database constraints are important for enforcing data integrity.

Application-level checks alone are not enough for concurrency-safe uniqueness.

---

# 112. Database Constraint vs Application Validation

Application:

```text
Check SKU exists
```

Database:

```text
UNIQUE(sku)
```

Two requests can pass the application check simultaneously.

The database constraint is the final authority.

Therefore:

```text
Application validation
+
Database constraints
```

should often work together.

---

# 113. Auditing

Spring Data JPA supports auditing features.

Common annotations:

```text
@CreatedDate
@LastModifiedDate
@CreatedBy
@LastModifiedBy
```

Example:

```java
@CreatedDate
private Instant createdAt;

@LastModifiedDate
private Instant updatedAt;
```

Enable auditing appropriately in the application.

---

# 114. @EntityListeners

Auditing can use:

```java
@EntityListeners(
    AuditingEntityListener.class
)
```

Example:

```java
@Entity
@EntityListeners(
    AuditingEntityListener.class
)
public class Product {

}
```

---

# 115. Soft Delete

Instead of physically deleting:

```sql
DELETE FROM products
```

you may use:

```text
active = false
```

or:

```text
deleted_at
```

Example:

```java
private boolean deleted;
```

Service:

```java
product.setDeleted(true);
```

Soft delete requires careful query design so deleted records are not accidentally returned.

---

# 116. Physical vs Soft Delete

Physical delete:

```text
Data removed
```

Soft delete:

```text
Data retained
but
treated as deleted
```

Soft delete can be useful for:

```text
Audit requirements
Recovery
Business history
Compliance
```

but increases query complexity and storage.

---

# 117. Auditing vs Soft Delete

Auditing answers:

```text
Who changed it?
When was it changed?
```

Soft delete answers:

```text
Should this record be treated as deleted?
```

They are separate concerns.

---

# 118. Database Migration

Do not rely on Hibernate schema generation for production database evolution in serious applications.

Common migration tools:

```text
Flyway
Liquibase
```

Typical process:

```text
Code change
↓
Migration script
↓
Database migration
↓
Application deployment
```

---

# 119. ddl-auto

Spring Boot can configure Hibernate schema behavior through:

```properties
spring.jpa.hibernate.ddl-auto=...
```

Common values:

```text
none
validate
update
create
create-drop
```

For production, avoid blindly using:

```text
update
```

for schema management.

Use proper migration tooling.

---

# 120. validate

A useful production-oriented approach is often:

```properties
spring.jpa.hibernate.ddl-auto=validate
```

This tells Hibernate to validate that the schema is compatible rather than modifying it.

The exact production strategy depends on the team's migration process.

---

# 121. N+1 Detection

Watch SQL logs during development/testing.

Useful tools and approaches:

```text
Hibernate SQL logging
Hibernate statistics
Database monitoring
APM
Integration tests
Query analysis
```

Do not optimize based only on intuition.

---

# 122. SQL Logging

Development configuration can enable relevant Hibernate SQL logging, but be careful with sensitive data.

For example, depending on the Hibernate/Spring Boot version:

```properties
logging.level.org.hibernate.SQL=DEBUG
```

Parameter logging should be used cautiously because it may expose sensitive values.

---

# 123. Connection Pooling

Spring Boot commonly uses:

```text
HikariCP
```

for JDBC connection pooling.

Conceptually:

```text
Application
    ↓
HikariCP
    ↓
Database connections
```

A connection pool avoids opening a new database connection for every request.

---

# 124. Connection Pool Configuration

Example:

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
```

Do not copy pool sizes blindly.

Choose values based on:

```text
Database capacity
Application concurrency
Request patterns
Number of application instances
Connection limits
```

---

# 125. Open Session in View

Spring Boot historically enables Open EntityManager in View for web applications by default in many setups.

Conceptually:

```text
HTTP request
    ↓
Persistence context remains available
    ↓
Controller/view accesses lazy relationship
```

This can hide poor query boundaries and lead to unexpected database access during response serialization.

For REST APIs, many teams prefer:

```properties
spring.jpa.open-in-view=false
```

and explicitly fetch the data required by the service/query layer.

The right setting depends on the architecture.

---

# 126. Entity Serialization Problem

Returning an entity directly can trigger lazy loading during JSON serialization:

```text
Controller
 ↓
Entity
 ↓
Jackson
 ↓
Lazy relationship accessed
 ↓
Unexpected query
```

This is another reason to return DTOs.

---

# 127. Bidirectional JSON Recursion

Suppose:

```text
Category
 ↓
Products
 ↓
Category
 ↓
Products
```

Serializing both sides directly can cause:

```text
Infinite recursion
```

Avoid exposing bidirectional entity graphs directly.

DTOs are usually the cleaner solution.

---

# 128. Repository Naming

Good:

```text
ProductRepository
OrderRepository
CustomerRepository
```

Avoid vague:

```text
DatabaseManager
DataHandler
CommonRepository
```

Repository should represent a clear persistence boundary.

---

# 129. Service + Repository Example

Repository:

```java
public interface ProductRepository
        extends JpaRepository<Product, Long> {

    List<Product>
    findByCategory(String category);
}
```

Service:

```java
@Service
public class ProductService {

    private final ProductRepository repository;

    public ProductService(
            ProductRepository repository) {

        this.repository = repository;
    }

    @Transactional(readOnly = true)
    public List<ProductResponse>
    getByCategory(String category) {

        return repository
            .findByCategory(category)
            .stream()
            .map(this::toResponse)
            .toList();
    }

    private ProductResponse toResponse(
            Product product) {

        return new ProductResponse(
            product.getId(),
            product.getName(),
            product.getPrice()
        );
    }
}
```

---

# 130. Repository Should Not Contain Business Logic

Bad:

```java
repository.calculateDiscount();
repository.validateOrder();
repository.checkUserPermission();
```

Business rules belong in the service/domain layer.

Repository should focus on persistence.

---

# 131. Service Should Not Build SQL Everywhere

Avoid putting:

```text
Raw SQL
Database-specific details
JPA query construction
```

throughout service methods.

Keep persistence logic inside:

```text
Repository
Specification
Query layer
```

as appropriate.

---

# 132. Repository Query Naming

Good:

```java
findByStatus(OrderStatus status)
```

Good:

```java
findByCustomerId(Long customerId)
```

Good:

```java
findByPriceBetween(
    BigDecimal min,
    BigDecimal max
)
```

Avoid excessively long derived method names.

---

# 133. Transactional Self-Invocation

Important interview topic:

```java
@Service
public class OrderService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {
    }
}
```

The internal call:

```text
methodA()
 ↓
methodB()
```

normally bypasses the Spring proxy.

Therefore the transactional interception on `methodB()` may not be applied as expected.

---

# 134. Transactional Proxy Mental Model

Think:

```text
Caller
  ↓
Spring Proxy
  ↓
@Transactional method
```

But:

```text
same object
  ↓
direct method call
```

can bypass the proxy.

This is a general Spring proxy concept.

---

# 135. Transaction Rollback

By default, Spring's declarative transaction behavior commonly rolls back for unchecked exceptions.

Checked exception behavior can be configured.

Example:

```java
@Transactional(
    rollbackFor = IOException.class
)
```

Do not assume every exception automatically causes rollback.

---

# 136. Read-Only Queries

For a read-only service:

```java
@Transactional(readOnly = true)
public ProductResponse get(
        Long id) {

    ...
}
```

This can communicate intent and allow provider/database optimizations.

---

# 137. Entity Equality

JPA entity equality can be tricky.

Avoid casually implementing:

```java
equals()
hashCode()
```

using mutable fields.

For entities with generated IDs, equality design should account for:

```text
Transient state
Generated identity
Proxies
Entity lifecycle
```

This is an advanced but important production concern.

---

# 138. Lombok and JPA

If using Lombok:

```java
@Data
@Entity
public class Product {

}
```

can cause problems because generated:

```text
equals
hashCode
toString
```

may traverse relationships or use mutable fields.

Prefer explicit Lombok annotations and carefully design entity equality and string representation.

---

# 139. Entity Constructors

JPA entities generally need a no-argument constructor with appropriate visibility.

Example:

```java
@Entity
public class Product {

    protected Product() {
    }

    public Product(
            String name,
            BigDecimal price) {

        this.name = name;
        this.price = price;
    }
}
```

Keep domain constructors meaningful while satisfying JPA requirements.

---

# 140. Entity Mutability

JPA entities are commonly mutable because the persistence provider tracks state changes.

But keep business invariants protected.

Instead of exposing unrestricted setters:

```java
product.setPrice(price);
```

a domain-oriented model may provide:

```java
product.changePrice(price);
```

when appropriate.

---

# 141. Large Object Fields

JPA supports:

```java
@Lob
private String description;
```

or binary data.

But storing large files directly in the database should be an architectural decision.

For large media, object storage is often more appropriate.

---

# 142. Embeddable Objects

JPA supports value objects using:

```java
@Embeddable
```

Example:

```java
@Embeddable
public class Address {

    private String city;

    private String country;
}
```

Then:

```java
@Embedded
private Address address;
```

This maps the value object's fields into the owning entity's table.

---

# 143. @Embedded

Example:

```java
@Entity
public class Customer {

    @Id
    private Long id;

    @Embedded
    private Address address;
}
```

Useful for logically grouped values that do not need their own table/entity identity.

---

# 144. Inheritance

JPA supports entity inheritance strategies:

```text
SINGLE_TABLE
JOINED
TABLE_PER_CLASS
```

Example:

```java
@Entity
@Inheritance(
    strategy = InheritanceType.SINGLE_TABLE
)
public abstract class Payment {

}
```

Choose inheritance carefully because it affects schema design and query performance.

---

# 145. Entity Graph Thinking

Before writing a query, ask:

```text
What data does this API actually need?
```

Then choose:

```text
Entity
DTO projection
Fetch join
EntityGraph
```

rather than loading an entire object graph by default.

---

# 146. Performance Checklist

```text
□ Avoid N+1 queries
□ Use pagination
□ Select only required data
□ Use DTO projections where useful
□ Index common query columns
□ Avoid unnecessary EAGER relationships
□ Monitor generated SQL
□ Keep transactions appropriately scoped
□ Tune connection pool based on workload
□ Avoid returning entities directly
□ Use batch operations carefully
□ Analyze slow queries with the database
```

---

# 147. Spring Data JPA Interview Questions

## What is JPA?

> JPA is a Java persistence specification that defines standard APIs and annotations for mapping Java objects to relational databases.

---

## What is Hibernate?

> Hibernate is a JPA implementation and ORM framework. Spring Boot commonly uses Hibernate underneath Spring Data JPA.

---

## What is Spring Data JPA?

> Spring Data JPA provides repository abstractions on top of JPA, reducing boilerplate CRUD and query code.

---

## What is an Entity?

> An entity is a Java class mapped to persistent data, usually a database table, and identified by a primary key.

---

## What is JpaRepository?

> `JpaRepository` provides common CRUD, pagination, sorting, and JPA-specific repository operations so we don't need to implement them manually.

---

## What is Dirty Checking?

> Dirty checking is the process where JPA detects changes to managed entities and synchronizes those changes with the database during flush or transaction commit.

---

## What is the Persistence Context?

> The persistence context is the set of managed entity instances associated with an EntityManager. It tracks entity state and acts as the first-level cache.

---

## What is Lazy Loading?

> Lazy loading delays loading an associated entity or collection until it is accessed. It can reduce unnecessary data retrieval, but the required persistence context/query design must be handled correctly.

---

## What is the N+1 Problem?

> N+1 occurs when one query loads the parent records and then an additional query is executed for each parent to load related data. It can create serious performance problems.

---

## How Do You Solve N+1?

> I would first identify the required data and then consider fetch joins, entity graphs, DTO projections, batch fetching, or a better query depending on the use case.

---

## What is @Transactional?

> `@Transactional` defines a transaction boundary around a method or class so multiple database operations can execute as one transactional unit according to the configured transaction rules.

---

## What is Optimistic Locking?

> Optimistic locking uses a version field, commonly `@Version`, to detect conflicting updates without holding a database lock throughout the transaction.

---

## Optimistic vs Pessimistic Locking?

> Optimistic locking assumes conflicts are relatively uncommon and detects them during update. Pessimistic locking obtains database-level locks to prevent conflicting operations while the transaction is active.

---

## JPQL vs Native SQL?

> JPQL operates on entities and their fields, while native SQL operates directly on database tables and columns. I prefer JPQL for database-independent entity queries and native SQL when database-specific functionality or complex SQL genuinely requires it.

---

## What Is @Modifying?

> `@Modifying` tells Spring Data that a repository query performs an update or delete rather than a normal select query.

---

## What Is @EntityGraph?

> `@EntityGraph` lets me specify which relationships should be fetched for a particular repository operation without changing the entity's default fetch strategy.

---

## Page vs Slice?

> `Page` provides pagination metadata including total counts, while `Slice` mainly tells me whether another slice exists. If total counts aren't needed, `Slice` can avoid some count-query overhead.

---

## Why Should We Avoid Returning Entities Directly?

> It couples the API to the persistence model, can expose internal fields, may trigger lazy loading, and can cause serialization problems. I prefer request and response DTOs.

---

# 148. Production Architecture

A clean Spring Boot JPA backend often looks like:

```text
controller/
    ProductController
    OrderController

service/
    ProductService
    OrderService

repository/
    ProductRepository
    OrderRepository

entity/
    Product
    Order
    Customer

dto/
    ProductRequest
    ProductResponse
    OrderRequest
    OrderResponse

mapper/
    ProductMapper
    OrderMapper

exception/
    GlobalExceptionHandler
    ProductNotFoundException

config/
    DatabaseConfig
```

---

# 149. Ecommerce Backend Example

For an ecommerce application:

```text
Product
Category
User
Cart
CartItem
Order
OrderItem
Payment
```

Possible relationships:

```text
Category
   ↓
Product

User
   ↓
Cart
   ↓
CartItem
   ↓
Product

User
   ↓
Order
   ↓
OrderItem
   ↓
Product
```

---

# 150. Recommended Ecommerce Entity Design

A simplified model:

```text
User
 ├── id
 ├── name
 └── email

Product
 ├── id
 ├── name
 ├── price
 └── category_id

Category
 ├── id
 └── name

Order
 ├── id
 ├── user_id
 ├── status
 └── total

OrderItem
 ├── id
 ├── order_id
 ├── product_id
 ├── quantity
 └── price_at_purchase
```

Notice:

```text
OrderItem
```

stores:

```text
price_at_purchase
```

rather than relying on the current product price.

That preserves historical order data.

---

# 151. Important Design Rule

Do not assume:

```text
Database model
=
API model
=
Domain model
```

They can be different.

A mature backend may have:

```text
Database Entity
       ↓
Domain/Business Model
       ↓
API DTO
```

The exact layering depends on project complexity.

---

# 152. Final Spring Data JPA Mental Model

```text
Repository
    ↓
Spring Data
    ↓
JPA
    ↓
Hibernate
    ↓
JDBC
    ↓
Database
```

Entity lifecycle:

```text
Transient
   ↓
Managed
   ↓
Detached
   ↓
Removed
```

Query options:

```text
Derived Query
     ↓
JPQL
     ↓
Native SQL
     ↓
Specification
     ↓
Projection
```

Performance:

```text
Pagination
Indexes
DTO Projection
Fetch Join
EntityGraph
Avoid N+1
Connection Pool
```

Transactions:

```text
Service
  ↓
@Transactional
  ↓
Repository operations
  ↓
Commit / Rollback
```

---

# 153. Spring Data JPA Checklist

```text
□ JPA
□ Hibernate
□ ORM
□ @Entity
□ @Table
□ @Id
□ @GeneratedValue
□ @Column
□ @Enumerated
□ @Transient
□ @Version
□ JpaRepository
□ CRUD methods
□ Derived queries
□ @Query
□ JPQL
□ Native queries
□ @Modifying
□ Persistence Context
□ Entity states
□ Dirty checking
□ Flush
□ EntityManager
□ Relationships
□ @OneToOne
□ @OneToMany
□ @ManyToOne
□ @ManyToMany
□ mappedBy
□ @JoinColumn
□ Cascade
□ Fetch types
□ Lazy loading
□ N+1
□ JOIN FETCH
□ @EntityGraph
□ DTO projections
□ Pagination
□ Sorting
□ Specifications
□ Transactions
□ Propagation
□ Isolation
□ Optimistic locking
□ Pessimistic locking
□ Auditing
□ Soft delete
□ Database indexes
□ Constraints
□ Flyway/Liquibase
□ ddl-auto
□ Connection pooling
```

---

# 154. Final Interview Rule

> **Spring Data JPA removes a lot of persistence boilerplate, but it does not remove the need to understand SQL, transactions, entity state, relationships, and query performance. In production, the most important JPA skills are designing the entity model correctly, keeping transaction boundaries clear, avoiding N+1 queries, fetching only the data you need, and understanding what Hibernate is doing underneath the repository abstraction.**

Next:

```text
01 Fundamentals
      ↓
02 Project Structure
      ↓
03 Dependency Injection & IoC
      ↓
04 Spring Beans & Configuration
      ↓
05 Spring Boot Annotations
      ↓
06 Configuration Properties & Profiles
      ↓
07 REST API Development
      ↓
08 Spring Data JPA
      ↓
09 Exception Handling & Validation
```
