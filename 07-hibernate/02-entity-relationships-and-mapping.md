# Hibernate & JPA — File 02: Entity Relationships & Mapping

This file covers how Hibernate/JPA maps relationships between entities.

Core topics:

```text
Entity Relationships
@ManyToOne
@OneToMany
@OneToOne
@ManyToMany
Foreign Keys
@JoinColumn
mappedBy
Owning Side
Inverse Side
Join Tables
Cascade
orphanRemoval
FetchType
Bidirectional Relationships
Unidirectional Relationships
Composite Keys
Embeddables
@Embedded
@Embeddable
@MapsId
Relationship Design
Common Mistakes
Interview Questions
Production Scenarios
```

---

# 1. Why Entity Relationships Matter

Real applications rarely contain isolated tables.

Example e-commerce system:

```text
User
 |
 +---- Order
          |
          +---- OrderItem
                    |
                    +---- Product
```

Hibernate lets us represent these database relationships using Java objects.

---

# 2. Database Relationship vs Java Relationship

Database:

```text
orders.customer_id
        ↓
customers.id
```

Java:

```java
class Order {
    private Customer customer;
}
```

Hibernate maps the relationship between the two models.

---

# 3. Four Main JPA Relationships

JPA supports:

```text
@OneToOne
@OneToMany
@ManyToOne
@ManyToMany
```

Memorize these.

---

# 4. @ManyToOne

Many records refer to one parent.

Example:

```text
Customer
   |
   +---- Order
   +---- Order
   +---- Order
```

Many Orders belong to one Customer.

Java:

```java
@ManyToOne
private Customer customer;
```

---

# 5. Database Representation of @ManyToOne

Usually:

```text
customers
----------------
id
name

orders
----------------
id
customer_id
total
```

The foreign key is normally on the `orders` table.

```text
orders.customer_id
        ↓
customers.id
```

---

# 6. @JoinColumn

`@JoinColumn` specifies the foreign-key column used for the relationship.

Example:

```java
@ManyToOne
@JoinColumn(name = "customer_id")
private Customer customer;
```

Database:

```text
orders
----------------
id
customer_id
```

---

# 7. Why @ManyToOne Is Important

It is one of the most common JPA relationships.

Examples:

```text
Order → Customer
OrderItem → Order
Employee → Department
Comment → Post
```

---

# 8. @OneToMany

One parent has many child entities.

Example:

```text
Customer
 |
 +---- Order
 +---- Order
 +---- Order
```

Java:

```java
@OneToMany
private List<Order> orders;
```

---

# 9. One-to-Many Usually Needs an Ownership Decision

A common mapping is:

```java
@OneToMany(mappedBy = "customer")
private List<Order> orders;
```

and:

```java
@ManyToOne
@JoinColumn(name = "customer_id")
private Customer customer;
```

Here:

```text
Order
→ owns the foreign key

Customer
→ inverse side
```

---

# 10. What Does mappedBy Mean?

`mappedBy` tells Hibernate:

> "This relationship is already mapped by the field on the other entity."

Example:

```java
@OneToMany(mappedBy = "customer")
private List<Order> orders;
```

The string:

```text
"customer"
```

must match the Java field in `Order`.

```java
@ManyToOne
private Customer customer;
```

---

# 11. Owning Side

The owning side controls the relationship mapping in the database.

For:

```text
Customer 1 ---- N Order
```

the typical owning side is:

```text
Order
```

because `orders.customer_id` contains the foreign key.

---

# 12. Inverse Side

The inverse side uses:

```java
mappedBy
```

Example:

```java
@OneToMany(mappedBy = "customer")
private List<Order> orders;
```

The inverse side does not independently own the foreign-key mapping.

---

# 13. Very Important Interview Rule

For a bidirectional relationship:

```text
One side
→ owns relationship

Other side
→ mappedBy
```

Do not define two independent mappings to the same relationship unless you intentionally want separate database relationships.

---

# 14. Bidirectional Relationship

Example:

```java
@Entity
class Customer {

    @OneToMany(mappedBy = "customer")
    private List<Order> orders;
}
```

```java
@Entity
class Order {

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
}
```

Now:

```text
Customer → Orders
Order → Customer
```

Both directions are available in Java.

---

# 15. Unidirectional Relationship

Only one entity knows about the relationship.

Example:

```java
@Entity
class Order {

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
}
```

Customer has no:

```java
List<Order>
```

This can be simpler when reverse navigation is not required.

---

# 16. Bidirectional vs Unidirectional

Unidirectional:

```text
Order → Customer
```

Bidirectional:

```text
Order ↔ Customer
```

Use bidirectional mappings only when the domain/use cases actually benefit from navigation in both directions.

---

# 17. Why Not Make Everything Bidirectional?

Bidirectional relationships can create:

```text
Complex object graphs
Serialization problems
Infinite recursion
Harder maintenance
Unexpected lazy loading
```

Example JSON problem:

```text
Customer
 ↓
Orders
 ↓
Customer
 ↓
Orders
 ↓
...
```

---

# 18. Jackson Infinite Recursion

A bidirectional relationship can cause JSON serialization loops.

Example:

```text
Customer
 → Order
   → Customer
     → Order
```

Solutions can include:

```text
DTOs
@JsonIgnore
@JsonManagedReference
@JsonBackReference
```

For REST APIs, DTOs are generally the cleaner long-term approach.

---

# 19. @OneToOne

One entity corresponds to one other entity.

Example:

```text
User
 |
 +---- UserProfile
```

Java:

```java
@OneToOne
@JoinColumn(name = "profile_id")
private UserProfile profile;
```

---

# 20. Database Representation of One-to-One

Could be:

```text
users
----------------
id
profile_id
```

with:

```text
profile_id → user_profiles.id
```

A unique constraint is often needed if the database must truly enforce one-to-one cardinality.

---

# 21. One-to-One Ownership

Example:

```java
@OneToOne
@JoinColumn(name = "profile_id", unique = true)
private UserProfile profile;
```

The `User` entity owns the relationship because it holds the foreign key.

---

# 22. Bidirectional One-to-One

Example:

```java
class User {
    @OneToOne
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}
```

```java
class UserProfile {
    @OneToOne(mappedBy = "profile")
    private User user;
}
```

---

# 23. @ManyToMany

Many entities relate to many other entities.

Example:

```text
Student
  ↕
Course
```

One Student:

```text
many Courses
```

One Course:

```text
many Students
```

---

# 24. Database Representation of Many-to-Many

Relational databases usually use a join table:

```text
students
---------
id

courses
---------
id

student_course
-------------
student_id
course_id
```

---

# 25. Basic Many-to-Many Mapping

```java
@ManyToMany
@JoinTable(
    name = "student_course",
    joinColumns = @JoinColumn(name = "student_id"),
    inverseJoinColumns = @JoinColumn(name = "course_id")
)
private Set<Course> courses;
```

---

# 26. Why Prefer Set for Many-to-Many?

A `Set` naturally represents unique relationships.

Example:

```text
Student 1
 ↓
Java
Java
Java
```

should generally result in one relationship row, not duplicate logical associations.

But collection choice should reflect actual domain semantics.

---

# 27. Many-to-Many Problems

Direct `@ManyToMany` can become difficult when the relationship itself has attributes.

Example:

```text
Student
Course
Enrollment
```

Suppose enrollment needs:

```text
enrolledAt
grade
status
```

A direct many-to-many mapping is no longer a good model.

---

# 28. Use an Explicit Join Entity

Instead:

```text
Student
   |
Enrollment
   |
Course
```

Java:

```java
@Entity
class Enrollment {

    @ManyToOne
    private Student student;

    @ManyToOne
    private Course course;

    private LocalDate enrolledAt;
    private String grade;
}
```

This is usually much more flexible.

---

# 29. E-Commerce Example

Avoid:

```text
Order
 ↕
Product
```

as a simple many-to-many if you need:

```text
quantity
priceAtPurchase
discount
```

Instead:

```text
Order
 |
 +---- OrderItem ---- Product
```

---

# 30. OrderItem

Example:

```java
@Entity
class OrderItem {

    @ManyToOne
    private Order order;

    @ManyToOne
    private Product product;

    private Integer quantity;

    private BigDecimal priceAtPurchase;
}
```

This is a very common backend design.

---

# 31. Why Store priceAtPurchase?

Product price can change.

Example:

```text
Product current price = ₹1200
```

But the customer purchased it at:

```text
₹999
```

The OrderItem should preserve:

```text
priceAtPurchase = ₹999
```

This is a domain modeling decision, not just a Hibernate mapping detail.

---

# 32. Cascade

Cascade controls whether persistence operations propagate from one entity to related entities.

Common options:

```text
PERSIST
MERGE
REMOVE
REFRESH
DETACH
ALL
```

---

# 33. CascadeType.PERSIST

When parent is persisted:

```java
entityManager.persist(order);
```

cascade persist can also persist its related entities.

---

# 34. CascadeType.MERGE

When an entity is merged:

```java
entityManager.merge(order);
```

the merge operation can cascade to related entities.

---

# 35. CascadeType.REMOVE

When the parent is removed:

```java
entityManager.remove(order);
```

related entities can also be removed.

Use carefully.

---

# 36. CascadeType.ALL

Means all cascade operations:

```text
PERSIST
MERGE
REMOVE
REFRESH
DETACH
```

It can be convenient, but don't apply it blindly.

---

# 37. Cascade Does Not Mean Database Cascade

This is a common interview question.

Hibernate/JPA cascade:

```text
Application persistence operation
```

Database cascade:

```text
Database foreign-key ON DELETE/UPDATE behavior
```

They are related concepts but not the same mechanism.

---

# 38. orphanRemoval

Example:

```java
@OneToMany(
    mappedBy = "order",
    orphanRemoval = true
)
private List<OrderItem> items;
```

If an OrderItem is removed from the managed collection, Hibernate can delete that orphan row.

---

# 39. Cascade REMOVE vs orphanRemoval

Cascade REMOVE:

```text
Parent deleted
 ↓
Child deleted
```

orphanRemoval:

```text
Child removed from relationship
 ↓
Child may be deleted
```

They solve related but different problems.

---

# 40. Aggregate Ownership

In domain modeling:

```text
Order
 |
 +---- OrderItem
```

OrderItem may exist only as part of an Order.

Then:

```text
Cascade
+
orphanRemoval
```

can make sense.

But don't use orphan removal when the child has an independent lifecycle.

---

# 41. Common Bad Mapping

```java
@OneToMany
private List<Order> orders;
```

without understanding ownership.

This can result in an additional join table or undesirable schema depending on the mapping.

Prefer an explicit ownership model.

---

# 42. Better One-to-Many Mapping

```java
@OneToMany(mappedBy = "customer")
private List<Order> orders;
```

and:

```java
@ManyToOne
@JoinColumn(name = "customer_id")
private Customer customer;
```

This clearly maps:

```text
orders.customer_id
```

---

# 43. FetchType

Relationships can define fetch behavior.

Common values:

```text
LAZY
EAGER
```

---

# 44. @ManyToOne Default Fetch

JPA specifies:

```text
@ManyToOne → EAGER by default
```

This is an important interview fact.

Many teams explicitly configure:

```java
@ManyToOne(fetch = FetchType.LAZY)
```

when appropriate.

---

# 45. @OneToMany Default Fetch

JPA specifies:

```text
@OneToMany → LAZY by default
```

This is generally safer for collections.

---

# 46. @OneToOne Default Fetch

JPA specifies:

```text
@OneToOne → EAGER by default
```

But eager loading should still be evaluated carefully.

---

# 47. @ManyToMany Default Fetch

JPA specifies:

```text
@ManyToMany → LAZY by default
```

---

# 48. Should You Trust Defaults?

For interview:

```text
Know the defaults.
```

For production:

```text
Choose fetch strategy intentionally.
```

Don't assume:

```text
LAZY = always good
EAGER = always bad
```

The correct choice depends on the use case.

---

# 49. Lazy Collection Example

```java
@OneToMany(
    mappedBy = "order",
    fetch = FetchType.LAZY
)
private List<OrderItem> items;
```

Loading:

```text
Order
```

doesn't necessarily load:

```text
OrderItems
```

until they are accessed.

---

# 50. Fetch Join

JPQL can explicitly fetch relationships:

```java
@Query("""
    select o
    from Order o
    join fetch o.items
    where o.id = :id
""")
Optional<Order> findOrderWithItems(Long id);
```

This can load the required data in one query.

Use carefully with large collections and pagination.

---

# 51. EntityGraph

Another way to define fetch plans:

```java
@EntityGraph(attributePaths = {"items"})
Optional<Order> findById(Long id);
```

This can be cleaner than writing JPQL for some use cases.

---

# 52. JoinColumn nullable

Example:

```java
@JoinColumn(
    name = "customer_id",
    nullable = false
)
```

This indicates the relationship is required at the database column level when schema generation/migration applies it.

---

# 53. Foreign Key Constraints

The database should enforce relationship integrity where appropriate.

Example:

```text
orders.customer_id
        ↓
customers.id
```

If a customer doesn't exist:

```text
Invalid order
```

A foreign key helps prevent this.

---

# 54. Optional Relationship

JPA supports:

```java
@ManyToOne(optional = false)
```

This communicates that the relationship is required at the object model level.

For database correctness, also consider:

```java
nullable = false
```

and an actual database constraint.

---

# 55. optional vs nullable

They operate at different levels.

```text
optional
→ JPA relationship semantics

nullable
→ Database column nullability
```

Use them consistently when the relationship is mandatory.

---

# 56. mappedBy Is a Field Name

Example:

```java
@OneToMany(mappedBy = "customer")
```

`customer` must be the Java property:

```java
private Customer customer;
```

It is not the database column name:

```text
customer_id
```

---

# 57. mappedBy Does Not Mean Column Name

Wrong:

```java
mappedBy = "customer_id"
```

Correct:

```java
mappedBy = "customer"
```

assuming the field is:

```java
private Customer customer;
```

---

# 58. JoinColumn vs mappedBy

Remember:

```text
@JoinColumn
→ defines the foreign-key mapping on the owning side

mappedBy
→ points to the owning-side Java property
```

---

# 59. Composite Keys

Sometimes an entity needs multiple columns as its identifier.

Example:

```text
student_id
course_id
```

Together identify:

```text
Enrollment
```

JPA supports approaches such as:

```text
@EmbeddedId
@IdClass
```

---

# 60. @EmbeddedId

Example:

```java
@Embeddable
public class EnrollmentId implements Serializable {

    private Long studentId;
    private Long courseId;
}
```

Then:

```java
@Entity
public class Enrollment {

    @EmbeddedId
    private EnrollmentId id;
}
```

---

# 61. @Embeddable

`@Embeddable` defines a value object whose fields are stored as columns of the owning entity.

Example:

```java
@Embeddable
class Address {
    private String city;
    private String state;
}
```

---

# 62. @Embedded

Use an embeddable object inside an entity:

```java
@Embedded
private Address address;
```

Database may contain:

```text
city
state
```

directly in the entity's table.

---

# 63. Embedded vs Entity

Embeddable:

```text
No independent identity
```

Entity:

```text
Has identity
```

Example:

```text
Address
→ @Embeddable

User
→ @Entity
```

---

# 64. Value Object Thinking

An embeddable is often appropriate when the object:

```text
Has no independent lifecycle
Is owned by the entity
Is conceptually part of the entity
```

Example:

```text
Money
Address
AuditInfo
```

---

# 65. @MapsId

`@MapsId` can be used when a relationship shares an identifier with the owning entity, commonly in dependent/composite-key mappings.

This is an advanced mapping feature and should be used when the database/domain model actually requires shared identity.

---

# 66. Collection Mapping

JPA supports collections such as:

```text
List
Set
Map
```

Examples:

```java
@OneToMany
private List<OrderItem> items;
```

---

# 67. List vs Set

List:

```text
Ordering matters
Duplicates may be meaningful
```

Set:

```text
Unique elements
```

Choose based on domain semantics rather than using one universally.

---

# 68. @OrderBy

Can specify ordering when loading a collection.

Example:

```java
@OneToMany(mappedBy = "order")
@OrderBy("createdAt DESC")
private List<OrderItem> items;
```

The exact SQL depends on provider/database.

---

# 69. Collection Size

Avoid blindly loading huge collections:

```text
Customer
 ↓
1 million Orders
```

Instead use:

```text
Pagination
Queries
Projections
Aggregations
```

---

# 70. Relationship Performance

A relationship mapping is not just an object-model decision.

Always consider:

```text
SQL generated
Number of queries
Join size
Indexes
Cardinality
Pagination
Memory
```

---

# 71. Index Foreign Keys

Foreign-key columns are frequently used in joins and filtering.

Example:

```text
orders.customer_id
```

may benefit from an index depending on workload and database design.

Don't assume Hibernate automatically creates the optimal index for every query.

---

# 72. Cascade + Large Collections

Danger:

```text
Customer
 ↓
1,000,000 Orders
```

with:

```text
cascade = ALL
```

A seemingly simple operation can trigger huge persistence work.

Understand cascade boundaries.

---

# 73. equals() and hashCode()

Entity equality is tricky.

This matters especially for:

```text
Set
HashMap
Hibernate proxies
Transient entities
Generated IDs
```

Avoid casually generating `equals()`/`hashCode()` over every field.

---

# 74. Generated ID Problem

Suppose:

```java
User user = new User();
```

Before persistence:

```text
id = null
```

After persistence:

```text
id = 100
```

If equality/hash code depends directly on a generated ID, behavior in hash-based collections can become surprising.

Entity equality should be designed deliberately.

---

# 75. Lombok Trap

Be careful with:

```java
@Data
```

on JPA entities.

Generated:

```text
equals()
hashCode()
toString()
```

can cause:

```text
Lazy loading
Circular references
Large object graphs
Proxy issues
```

Prefer deliberate methods for entities.

---

# 76. toString() Trap

Bad:

```java
@Override
public String toString() {
    return "Customer{orders=" + orders + "}";
}
```

If `orders` is lazy:

```text
toString()
 ↓
Lazy loading
 ↓
Database query
```

or potentially:

```text
Circular reference
```

Keep entity `toString()` simple.

---

# 77. DTOs and Relationships

Don't automatically return entities directly from REST APIs.

Instead:

```text
Entity
 ↓
Service
 ↓
DTO
 ↓
Controller
 ↓
JSON
```

Benefits:

```text
Avoid circular references
Control payload size
Prevent accidental lazy loading
Hide internal fields
Stable API contract
```

---

# 78. Example DTO

```java
public record OrderResponse(
    Long id,
    BigDecimal total,
    List<OrderItemResponse> items
) {}
```

Map only what the API actually needs.

---

# 79. Relationship Mapping in E-Commerce

A good model:

```text
Customer
   |
   +---- Order
             |
             +---- OrderItem ---- Product
```

Mappings:

```text
Customer 1 → N Order

Order 1 → N OrderItem

Product 1 → N OrderItem
```

This avoids a direct Order ↔ Product many-to-many when relationship attributes exist.

---

# 80. Example E-Commerce Mapping

```java
@Entity
class Order {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;

    @OneToMany(
        mappedBy = "order",
        cascade = CascadeType.ALL,
        orphanRemoval = true
    )
    private List<OrderItem> items = new ArrayList<>();
}
```

This is a common aggregate-style mapping.

---

# 81. Synchronize Both Sides

For bidirectional relationships, helper methods can keep both sides consistent.

Example:

```java
public void addItem(OrderItem item) {
    items.add(item);
    item.setOrder(this);
}
```

And:

```java
public void removeItem(OrderItem item) {
    items.remove(item);
    item.setOrder(null);
}
```

This prevents the Java object graph from becoming inconsistent.

---

# 82. Why Helper Methods Matter

Without:

```java
item.setOrder(this);
```

you may have:

```text
Order.items contains item
```

but:

```text
item.order = null
```

The database relationship is controlled by the owning side.

Keep both sides synchronized in memory.

---

# 83. Owning Side Example

```text
Order
  |
  | mappedBy = "order"
  ↓
OrderItem
```

Here:

```text
OrderItem.order
```

owns the relationship.

Therefore:

```java
item.setOrder(order);
```

is important.

---

# 84. Common Mistake

Developer writes:

```java
order.getItems().add(item);
```

but doesn't set:

```java
item.setOrder(order);
```

Result:

```text
In-memory relationship
≠
Owning-side relationship
```

The foreign key may not be updated as expected.

---

# 85. orphanRemoval Example

```java
order.removeItem(item);
```

If:

```java
orphanRemoval = true
```

and the child is managed, Hibernate can delete the corresponding row.

This is useful when:

```text
OrderItem cannot exist without Order
```

---

# 86. When Not to Use orphanRemoval

Don't use it when:

```text
Child has independent lifecycle
Child may belong elsewhere
Removing association should not delete the entity
```

Example:

```text
Product
```

should not be deleted merely because an OrderItem relationship changes.

---

# 87. Many-to-Many Cascade Warning

Avoid blindly using:

```java
@ManyToMany(cascade = CascadeType.ALL)
```

because removing one entity can have unintended effects on shared entities.

For shared entities, lifecycle ownership is usually different.

---

# 88. Shared Entity Example

```text
Student A
   |
   +→ Java Course
   |
Student B
   |
   +→ Java Course
```

Deleting Student A should not delete:

```text
Java Course
```

because Student B still uses it.

This is why cascade remove on many-to-many is dangerous.

---

# 89. Database First vs Entity First

Two common approaches:

```text
Database-first
Entity model derived from existing schema

Code-first
Entities drive schema generation/migrations
```

In production, schema changes should generally be managed through migration tools rather than relying blindly on automatic schema generation.

Examples:

```text
Flyway
Liquibase
```

---

# 90. ddl-auto

Spring Boot commonly exposes:

```properties
spring.jpa.hibernate.ddl-auto=
```

Possible values include:

```text
none
validate
update
create
create-drop
```

---

# 91. Production Recommendation

Avoid relying on:

```text
update
```

as your production database migration strategy.

Prefer:

```text
Flyway
or
Liquibase
```

for controlled, versioned schema changes.

---

# 92. Relationship Schema Evolution

Changing:

```text
@OneToMany
```

to:

```text
@ManyToMany
```

is not merely a Java change.

It may require:

```text
New table
Data migration
Indexes
Foreign keys
Application compatibility
```

ORM mappings must match actual database design.

---

# 93. Interview Question

### "What is the owning side of a JPA relationship?"

Answer:

> "The owning side is the side responsible for managing the relationship mapping, usually the side containing the foreign key. In a typical Customer-One-to-Many-Order relationship, Order owns the relationship because orders.customer_id references Customer."

---

# 94. Interview Question

### "What does mappedBy mean?"

Answer:

> "`mappedBy` tells JPA that the relationship is already mapped by a property on the other entity. It identifies the Java property on the owning side, not the database column name."

---

# 95. Interview Question

### "What is @JoinColumn?"

Answer:

> "`@JoinColumn` defines the database column used to join entities, commonly the foreign-key column on the owning side."

---

# 96. Interview Question

### "Unidirectional vs bidirectional?"

Answer:

> "A unidirectional relationship allows navigation from one entity to another, while a bidirectional relationship allows navigation in both directions. I prefer bidirectional mappings only when the domain actually requires both directions."

---

# 97. Interview Question

### "What is cascade?"

Answer:

> "Cascade controls whether persistence operations such as persist, merge or remove are propagated from one entity to related entities. It is a JPA lifecycle concept and is not the same as database cascade."

---

# 98. Interview Question

### "Cascade REMOVE vs orphanRemoval?"

Answer:

> "Cascade REMOVE generally propagates parent removal to the child. orphanRemoval can delete a child when it is removed from the parent's relationship, assuming the mapping supports that behavior."

---

# 99. Interview Question

### "Why is @ManyToOne often better than @ManyToMany?"

Answer:

> "Many-to-many becomes difficult when the relationship has its own attributes such as quantity, price or status. In those cases I model the relationship as an explicit entity, such as OrderItem."

---

# 100. Interview Question

### "Why use DTOs with JPA entities?"

Answer:

> "DTOs let us control the API contract and avoid exposing persistence details. They also help prevent circular serialization, unexpected lazy loading and unnecessarily large response payloads."

---

# 101. Interview Question

### "Why can bidirectional relationships cause JSON recursion?"

Answer:

> "Because each entity holds a reference to the other. A serializer can follow Customer → Orders → Customer indefinitely. DTOs are usually the cleanest way to control the API representation."

---

# 102. Interview Question

### "What are the default fetch types?"

Answer:

> "JPA specifies EAGER for ManyToOne and OneToOne, and LAZY for OneToMany and ManyToMany. In production, I still choose fetch plans intentionally rather than relying blindly on defaults."

---

# 103. Interview Question

### "What is @Embeddable?"

Answer:

> "`@Embeddable` defines a value type whose fields are persisted as columns in the owning entity's table. Unlike an entity, it does not have an independent identity."

---

# 104. Interview Question

### "When would you use @EmbeddedId?"

Answer:

> "I'd use `@EmbeddedId` when an entity has a composite identifier represented by an embeddable value object, such as an enrollment identified by studentId and courseId."

---

# 105. Interview Scenario

### "Customer has 100,000 orders. Should you load customer.getOrders()?"

Answer:

> "No, not blindly. I'd use a paginated query or a specific query for the required order data. Loading a huge collection can create high memory usage and database overhead."

---

# 106. Interview Scenario

### "Why isn't the foreign key updated?"

Check:

```text
Owning side
mappedBy
Relationship synchronization
Transaction
Cascade
```

If:

```text
Order.items.add(item)
```

but:

```text
item.order
```

isn't set, the owning side may not contain the expected relationship.

---

# 107. Interview Scenario

### "Deleting an Order also deletes OrderItems. Is that okay?"

Answer:

> "If OrderItem is an owned child with no independent lifecycle, cascade remove and orphanRemoval can be appropriate. If OrderItem has an independent lifecycle, I would avoid automatic deletion."

---

# 108. Interview Scenario

### "A Product disappears after deleting an Order."

Possible problem:

```text
Cascade REMOVE
```

on a relationship involving a shared Product entity.

Review:

```text
Lifecycle ownership
Cascade configuration
Many-to-many mappings
```

---

# 109. Interview Scenario

### "API response causes StackOverflowError."

Likely:

```text
Bidirectional relationship
+
Recursive JSON serialization
```

Fix:

```text
DTOs
```

or carefully controlled serialization annotations.

---

# 110. Interview Scenario

### "Adding an OrderItem doesn't update the database."

Check:

```text
Which side owns relationship?
Did item.setOrder(order) happen?
Is transaction active?
Is entity managed?
Is flush occurring?
```

---

# 111. Best Practices

```text
Prefer explicit ownership
Use mappedBy correctly
Use LAZY where appropriate
Don't blindly use EAGER
Avoid unnecessary bidirectional mappings
Use DTOs at API boundaries
Use helper methods for bidirectional collections
Use cascade based on lifecycle ownership
Use orphanRemoval only for true owned children
Prefer explicit join entities when relationships have attributes
Index foreign keys where appropriate
Paginate large collections
Understand generated SQL
Use database constraints
Use Flyway/Liquibase for production schema evolution
```

---

# 112. Relationship Mental Model

Remember:

```text
Database:

Parent
  |
  | PK
  ↓
Child
  |
  | FK
  ↓

JPA:

Parent
  |
  | @OneToMany(mappedBy = "parent")
  ↓
Child
  |
  | @ManyToOne
  | @JoinColumn(...)
  ↓
Parent
```

The child commonly owns the foreign-key mapping.

---

# 113. E-Commerce Mental Model

```text
Customer
   |
   | 1
   |
   | N
   ↓
Order
   |
   | 1
   |
   | N
   ↓
OrderItem
   |
   | N
   |
   | 1
   ↓
Product
```

This is often better than:

```text
Order ↔ Product
```

because:

```text
OrderItem
```

can contain:

```text
quantity
priceAtPurchase
discount
```

---

# 114. Final Interview Answer

If asked:

> "Explain how you would model relationships in a Spring Boot e-commerce application."

Use:

> "I'd model Customer to Order as one-to-many, Order to OrderItem as one-to-many, and Product to OrderItem as many-to-one. The foreign-key side would own the relationship, while the parent collection would normally use `mappedBy`. For Order and OrderItem, cascade and orphanRemoval can be appropriate if OrderItem has no independent lifecycle. I'd keep large collections lazy and fetch the exact data needed by each use case. At the REST boundary I'd use DTOs rather than exposing entities directly, and I'd make sure the database has proper foreign keys and indexes."

---

# 115. Revision Checklist

```text
□ @ManyToOne
□ @OneToMany
□ @OneToOne
□ @ManyToMany
□ @JoinColumn
□ mappedBy
□ Owning side
□ Inverse side
□ Foreign keys
□ Unidirectional relationships
□ Bidirectional relationships
□ Join tables
□ Cascade
□ CascadeType.PERSIST
□ CascadeType.MERGE
□ CascadeType.REMOVE
□ CascadeType.ALL
□ orphanRemoval
□ FetchType.LAZY
□ FetchType.EAGER
□ Default fetch types
□ N+1
□ JOIN FETCH
□ EntityGraph
□ Composite keys
□ @EmbeddedId
□ @IdClass
□ @Embeddable
□ @Embedded
□ @MapsId
□ List vs Set
□ @OrderBy
□ equals/hashCode
□ Lombok pitfalls
□ DTOs
□ Large collections
□ Foreign-key indexes
□ Schema migrations
□ Flyway
□ Liquibase
□ Relationship synchronization
□ E-commerce mapping
□ Interview scenarios
```

---

# 116. What Comes Next

Next:

```text
File 03 → Fetching, Lazy/Eager Loading & N+1
```

We'll go much deeper into:

```text
LazyInitializationException
JOIN FETCH
EntityGraph
N+1 detection
Batch fetching
Pagination
Fetch joins with collections
MultipleBagFetchException
DTO projections
Open Session in View
Performance trade-offs
```

The key interview lesson is:

> **JPA relationships are not just Java annotations. They define how object graphs map to foreign keys, joins, collections and database operations. Always think about ownership, lifecycle, fetch strategy and the SQL that the mapping will produce.**
