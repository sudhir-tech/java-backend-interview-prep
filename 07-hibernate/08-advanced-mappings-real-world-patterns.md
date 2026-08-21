# Hibernate & JPA — File 08: Advanced Mappings & Real-World Patterns

This file covers advanced Hibernate/JPA mapping concepts that commonly appear in experienced Java backend interviews and real production systems.

Core topics:

```text
Inheritance
SINGLE_TABLE
JOINED
TABLE_PER_CLASS
@MappedSuperclass
Embeddables
Composite Keys
@EmbeddedId
@IdClass
@MapsId
Auditing
@CreatedDate
@LastModifiedDate
Entity Listeners
Enums
@Enumerated
AttributeConverter
UUIDs
Natural IDs
Soft Delete
Hibernate Filters / Restrictions
Database Constraints
JSON Columns
Custom Types
E-commerce Mapping
Real-World Design
Interview Questions
Production Scenarios
```

---

# 1. Why Advanced Mapping Matters

Basic JPA mapping is:

```text
Java class
 ↓
Database table
```

Real applications often have:

```text
Inheritance
Value objects
Composite identifiers
Audit fields
Enums
JSON
Soft deletion
Legacy schemas
Database-specific types
```

Hibernate provides tools to map these models.

---

# 2. Entity Inheritance

Suppose:

```text
Payment
 ├── CardPayment
 ├── UpiPayment
 └── BankTransferPayment
```

Java inheritance is:

```java
abstract class Payment {
}
```

Hibernate needs to know how this hierarchy should be represented in the database.

Common strategies:

```text
SINGLE_TABLE
JOINED
TABLE_PER_CLASS
```

---

# 3. SINGLE_TABLE

Example:

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public abstract class Payment {

    @Id
    private Long id;

    private BigDecimal amount;
}
```

Database:

```text
payments
------------------------------------------------
id | type | amount | cardNumber | upiId | bank
```

All subclasses share one table.

---

# 4. Discriminator Column

SINGLE_TABLE normally uses a discriminator.

Example:

```java
@DiscriminatorColumn(name = "payment_type")
```

Subclasses:

```java
@Entity
@DiscriminatorValue("CARD")
class CardPayment extends Payment {
}
```

and:

```java
@Entity
@DiscriminatorValue("UPI")
class UpiPayment extends Payment {
}
```

Database:

```text
payment_type
-------------
CARD
UPI
BANK
```

---

# 5. SINGLE_TABLE Advantages

```text
Simple schema
Fast polymorphic queries
Fewer joins
Good read performance
```

---

# 6. SINGLE_TABLE Disadvantages

Potential problems:

```text
Many nullable columns
Large table
Subtype-specific fields mixed together
Database constraints can become harder
```

Example:

```text
CardPayment needs card_number
UPI needs upi_id
BankTransfer needs bank_account
```

Many columns may be NULL for unrelated subtypes.

---

# 7. JOINED Strategy

Example:

```java
@Inheritance(strategy = InheritanceType.JOINED)
```

Database:

```text
payment
----------------
id
amount

card_payment
----------------
id
card_number

upi_payment
----------------
id
upi_id
```

The subclass table shares the same primary key.

---

# 8. JOINED Query

Loading a subclass may require:

```text
JOIN
```

between:

```text
payment
+
card_payment
```

Polymorphic queries can therefore involve more joins.

---

# 9. JOINED Advantages

```text
Normalized schema
Fewer NULL columns
Subtype-specific fields stay in subtype tables
Database structure is cleaner
```

---

# 10. JOINED Disadvantages

```text
More joins
Potentially more complex SQL
Polymorphic reads can be slower
```

---

# 11. TABLE_PER_CLASS

Each concrete subclass gets its own table.

Example:

```text
card_payment
----------------
id
amount
card_number

upi_payment
----------------
id
amount
upi_id
```

There may be no shared physical parent table.

---

# 12. TABLE_PER_CLASS Trade-Off

A polymorphic query may require:

```text
UNION
```

across subclass tables.

This can become expensive as the hierarchy grows.

---

# 13. Inheritance Strategy Comparison

```text
SINGLE_TABLE
→ One table
→ Fast reads
→ Nullable columns

JOINED
→ Normalized tables
→ More joins

TABLE_PER_CLASS
→ Separate concrete tables
→ Polymorphic queries can require UNION
```

---

# 14. Which Strategy Should You Choose?

Ask:

```text
How many subclasses?
How frequently are polymorphic queries?
How many subtype-specific fields?
How important are database constraints?
How large is the dataset?
```

Don't choose based only on personal preference.

---

# 15. @MappedSuperclass

`@MappedSuperclass` is different from entity inheritance.

Example:

```java
@MappedSuperclass
public abstract class BaseEntity {

    @Id
    @GeneratedValue
    private Long id;

    private Instant createdAt;
}
```

Then:

```java
@Entity
class Product extends BaseEntity {
}
```

and:

```java
@Entity
class Order extends BaseEntity {
}
```

---

# 16. What @MappedSuperclass Means

It provides:

```text
Reusable mapping fields
```

to subclasses.

It does not represent a standalone entity table by itself.

---

# 17. @MappedSuperclass vs @Inheritance

`@Inheritance`:

```text
Represents an entity inheritance hierarchy.
```

`@MappedSuperclass`:

```text
Shares mapped fields with entities.
```

Typical use:

```text
id
createdAt
updatedAt
createdBy
updatedBy
```

---

# 18. Embeddables

Embeddables represent value objects.

Example:

```java
@Embeddable
public class Address {

    private String street;
    private String city;
    private String postalCode;
}
```

Entity:

```java
@Embedded
private Address address;
```

---

# 19. Embeddable Database Mapping

Instead of:

```text
address table
```

the fields may be stored directly in the owner table:

```text
customer
--------------------------------
id
street
city
postal_code
```

---

# 20. Why Use Embeddables?

Good for:

```text
Address
Money
Coordinates
Contact information
Audit metadata
Value objects
```

They model data that has no independent identity.

---

# 21. Entity vs Value Object

Entity:

```text
Has identity
Lifecycle
```

Value object:

```text
Defined by its values
Usually owned by another entity
```

Example:

```text
Customer → Entity
Address → Value Object
```

---

# 22. Attribute Overrides

If an entity has two addresses:

```java
@Embedded
private Address billingAddress;

@Embedded
private Address shippingAddress;
```

both would otherwise map to the same column names.

Use:

```java
@AttributeOverrides({
    @AttributeOverride(
        name = "city",
        column = @Column(name = "billing_city")
    )
})
```

and corresponding overrides for shipping fields.

---

# 23. Composite Keys

Sometimes an entity is uniquely identified by multiple columns.

Example:

```text
order_id
product_id
```

Together:

```text
OrderItem identity
```

---

# 24. @EmbeddedId

Example:

```java
@Embeddable
public class OrderItemId {

    private Long orderId;
    private Long productId;
}
```

Entity:

```java
@Entity
public class OrderItem {

    @EmbeddedId
    private OrderItemId id;
}
```

---

# 25. @IdClass

Alternative approach:

```java
@Entity
@IdClass(OrderItemId.class)
public class OrderItem {

    @Id
    private Long orderId;

    @Id
    private Long productId;
}
```

---

# 26. @EmbeddedId vs @IdClass

`@EmbeddedId`:

```text
Composite key is represented as an embeddable object.
```

`@IdClass`:

```text
ID fields remain directly on the entity.
```

Both are standard JPA approaches.

---

# 27. Composite Key Requirements

The key class should follow JPA requirements such as:

```text
Serializable
Proper equals()
Proper hashCode()
```

The exact implementation depends on whether `@EmbeddedId` or `@IdClass` is used.

---

# 28. @MapsId

`@MapsId` is useful when an entity relationship participates in the entity's identifier.

A common case:

```text
Order
 ↓
OrderItem
```

where the item's identity includes the order ID.

It helps map:

```text
Relationship
+
Identifier
```

without duplicating the same identity state incorrectly.

---

# 29. Auditing

Most production entities need fields such as:

```text
createdAt
updatedAt
createdBy
updatedBy
```

Spring Data JPA supports auditing.

Example:

```java
@EntityListeners(AuditingEntityListener.class)
@Entity
class Product {

    @CreatedDate
    private Instant createdAt;

    @LastModifiedDate
    private Instant updatedAt;
}
```

---

# 30. Enable Auditing

Spring configuration commonly includes:

```java
@EnableJpaAuditing
```

Then Spring Data can populate auditing fields.

---

# 31. CreatedDate

Example:

```java
@CreatedDate
private Instant createdAt;
```

It represents when the entity was created.

---

# 32. LastModifiedDate

Example:

```java
@LastModifiedDate
private Instant updatedAt;
```

It represents the latest modification timestamp according to the auditing mechanism.

---

# 33. CreatedBy / LastModifiedBy

Spring Data can also support:

```java
@CreatedBy
private String createdBy;

@LastModifiedBy
private String updatedBy;
```

An `AuditorAware` implementation supplies the current actor.

---

# 34. Entity Listener

JPA entity listeners can react to lifecycle events.

Examples:

```text
@PrePersist
@PreUpdate
@PreRemove
@PostLoad
@PostPersist
@PostUpdate
@PostRemove
```

---

# 35. @PrePersist

Example:

```java
@PrePersist
public void beforeInsert() {
    ...
}
```

Called before the entity is persisted.

Useful for:

```text
Default values
Generated application metadata
```

---

# 36. @PreUpdate

Called before an entity update is persisted.

Be careful about putting complex business logic into entity lifecycle callbacks.

---

# 37. Why Avoid Heavy Listener Logic?

Entity listeners can make behavior:

```text
Implicit
Hard to trace
Hard to test
```

Business workflows usually belong in:

```text
Service/domain layer
```

rather than hidden inside persistence callbacks.

---

# 38. Enum Mapping

Java:

```java
public enum OrderStatus {
    PENDING,
    PAID,
    CANCELLED
}
```

JPA:

```java
@Enumerated(EnumType.STRING)
private OrderStatus status;
```

---

# 39. STRING vs ORDINAL

Avoid:

```java
@Enumerated(EnumType.ORDINAL)
```

for most production applications.

Ordinal stores:

```text
0
1
2
```

Changing enum order can corrupt the meaning of existing data.

---

# 40. Prefer EnumType.STRING

```java
@Enumerated(EnumType.STRING)
private OrderStatus status;
```

Database:

```text
PENDING
PAID
CANCELLED
```

This is more stable and readable.

---

# 41. Enum String Trade-Off

String values can:

```text
Take more storage
Require careful renaming
```

But they are generally safer than ordinal values for persisted business state.

---

# 42. AttributeConverter

Use `AttributeConverter` when Java and database representations differ.

Example:

```java
@Converter
public class BooleanConverter
        implements AttributeConverter<Boolean, String> {
}
```

---

# 43. Converter Example

Java:

```text
true / false
```

Database:

```text
Y / N
```

Converter:

```java
@Converter
public class YesNoConverter
        implements AttributeConverter<Boolean, String> {

    @Override
    public String convertToDatabaseColumn(Boolean value) {
        return Boolean.TRUE.equals(value) ? "Y" : "N";
    }

    @Override
    public Boolean convertToEntityAttribute(String value) {
        return "Y".equals(value);
    }
}
```

---

# 44. When Use AttributeConverter?

Useful for:

```text
Legacy schema
Custom enum representation
Boolean flags
Encrypted/encoded values
Value-object conversion
```

Be careful with security-sensitive transformations such as encryption: key management, searchability and rotation need architectural consideration.

---

# 45. UUIDs

UUIDs can be used as identifiers.

Example:

```java
@Id
@GeneratedValue
private UUID id;
```

Exact mapping/generation depends on Hibernate version and database.

---

# 46. UUID Advantages

```text
Globally unique
Can be generated without central coordination
Useful in distributed systems
Harder to guess than sequential IDs
```

---

# 47. UUID Trade-Offs

Potential drawbacks:

```text
Larger indexes
Larger storage
Random insertion patterns
Potential index fragmentation
Less human-readable
```

Some systems use time-ordered UUID variants or application/database-specific strategies to improve index locality.

---

# 48. Natural ID

A natural ID is a business identifier.

Examples:

```text
email
ISBN
SKU
employeeNumber
```

Example:

```text
Product SKU = SKU-12345
```

This is different from a surrogate database ID.

---

# 49. Surrogate vs Natural Key

Surrogate:

```text
id = 12345
```

Natural:

```text
sku = SKU-12345
```

A business key can change; a surrogate ID usually should not.

---

# 50. Natural ID Considerations

If a natural business identifier is supposed to be unique:

```text
Add a database UNIQUE constraint.
```

Application-level checking alone is not enough under concurrency.

---

# 51. Database Constraints

Use database constraints to protect data integrity.

Examples:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
```

Hibernate validation is useful, but database constraints remain important.

---

# 52. Why Database Constraints Matter

Suppose two requests execute:

```text
Check email doesn't exist
 ↓
Both see "not found"
 ↓
Both insert
```

Without:

```text
UNIQUE(email)
```

duplicates can occur.

The database must enforce the invariant.

---

# 53. Soft Delete

Instead of:

```sql
DELETE FROM product
```

store:

```text
deleted = true
```

or:

```text
deleted_at = timestamp
```

---

# 54. Soft Delete Advantages

```text
Recoverability
Audit history
Historical reporting
Avoid physical deletion
```

---

# 55. Soft Delete Disadvantages

Every query must correctly exclude deleted rows.

Potential problems:

```text
Forgotten filter
Unique constraints
Index growth
Storage growth
Relationship semantics
```

---

# 56. Soft Delete with Hibernate

Hibernate versions provide provider-specific mechanisms for restricting entity queries.

For example, newer Hibernate versions support features such as:

```text
@SQLRestriction
```

for static SQL restrictions.

Exact annotations and behavior are Hibernate-version specific.

---

# 57. Hibernate Filters

Hibernate also provides filters for dynamic restrictions.

Conceptually:

```text
Tenant filter
Active records
Date-based visibility
```

Filters are powerful but provider-specific.

---

# 58. Soft Delete Best Practice

If using soft delete:

```text
Define one consistent strategy
Index deleted/status columns when useful
Test every important query
Handle unique constraints
Define relationship behavior
Plan archival
```

Don't treat it as simply:

```text
deleted = true
```

---

# 59. Multi-Tenancy

Advanced applications may have:

```text
Tenant A
Tenant B
Tenant C
```

Data isolation can be implemented using:

```text
tenant_id column
separate schemas
separate databases
```

Hibernate has provider-specific multi-tenancy capabilities.

---

# 60. tenant_id Column

A shared table can look like:

```text
orders
--------------------------------
id
tenant_id
customer_id
total
```

Queries must ensure:

```text
tenant_id = currentTenant
```

Security is critical here.

---

# 61. Multi-Tenant Risk

A missing tenant predicate can cause:

```text
Tenant A
 ↓
sees Tenant B data
```

This is a severe security issue.

Tenant isolation should be enforced through multiple layers where appropriate:

```text
Application
ORM/query layer
Database
```

---

# 62. JSON Columns

Modern databases can support JSON columns.

Example concept:

```text
product
-------------------------
id
name
attributes JSON
```

Useful for:

```text
Flexible attributes
Metadata
External payload fragments
```

---

# 63. JSON Column Trade-Off

JSON is useful when structure is flexible, but don't use it for everything.

Relational columns are usually better when data is:

```text
Frequently filtered
Joined
Indexed individually
Strongly constrained
```

---

# 64. JSON + Hibernate

Hibernate versions provide different ways to map JSON.

Depending on Hibernate/database:

```text
JSON mapping annotations
Custom types
Attribute converters
```

The exact mechanism is provider/version dependent.

---

# 65. Custom Types

Hibernate can support custom mappings for database-specific types.

Examples:

```text
JSON
ARRAY
Geospatial types
Database-specific types
```

Prefer standard JPA mechanisms when sufficient.

Use Hibernate-specific types when the database capability is important.

---

# 66. Real-World E-commerce Model

A possible model:

```text
User
 |
 +--- Address
 |
 +--- Orders
       |
       +--- OrderItems
       |      |
       |      +--- Product
       |
       +--- Payment
```

---

# 67. OrderItem as an Entity

Don't model order items only as:

```text
@ManyToMany
```

if you need:

```text
quantity
unitPrice
discount
tax
```

Instead:

```text
Order
  |
  +--- OrderItem
          |
          +--- Product
```

This is an association entity.

---

# 68. Snapshot Pricing

Important e-commerce design:

When an order is placed:

```text
Product current price = ₹1,000
```

OrderItem should store:

```text
unitPrice = ₹1,000
```

Why?

Because the product price may later change:

```text
Product price = ₹1,200
```

The historical order should still show:

```text
₹1,000
```

This is a domain-model decision, not merely a Hibernate mapping detail.

---

# 69. Order Status

Use:

```java
@Enumerated(EnumType.STRING)
private OrderStatus status;
```

Possible:

```text
PENDING
CONFIRMED
SHIPPED
DELIVERED
CANCELLED
```

---

# 70. Audit Fields

Base entity:

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @Id
    @GeneratedValue
    private Long id;

    @CreatedDate
    private Instant createdAt;

    @LastModifiedDate
    private Instant updatedAt;
}
```

Then:

```text
Order extends BaseEntity
Product extends BaseEntity
User extends BaseEntity
```

---

# 71. Address as Embeddable

```java
@Embeddable
public class Address {

    private String line1;
    private String city;
    private String state;
    private String postalCode;
}
```

Use:

```java
@Embedded
private Address address;
```

This keeps value-object modeling clean.

---

# 72. Payment Inheritance

If payment types have substantially different data:

```text
Payment
 ├── CardPayment
 ├── UpiPayment
 └── BankTransferPayment
```

Evaluate:

```text
SINGLE_TABLE
JOINED
```

based on query patterns and schema requirements.

---

# 73. Production Mapping Principle

Ask:

```text
Is this an Entity?
Is this a Value Object?
Does it have independent lifecycle?
Does it need its own table?
Will it be queried independently?
How large can the relationship become?
```

Don't map classes mechanically.

---

# 74. Entity Relationship Principle

A relationship should reflect business semantics.

Example:

```text
Customer → Address
```

may be:

```text
@Embedded
```

if the address is a value object.

But:

```text
Customer → Orders
```

should normally be an entity relationship because orders have independent identity and lifecycle.

---

# 75. Common Mapping Mistake

Bad:

```text
Use @ManyToMany everywhere
```

Many-to-many often hides important relationship data.

If the relationship has:

```text
quantity
createdAt
status
price
metadata
```

use an association entity.

---

# 76. Common Mapping Mistake

Bad:

```text
Use inheritance simply because Java classes have inheritance.
```

Database inheritance can add:

```text
joins
unions
nullable columns
complex queries
```

Choose based on persistence requirements.

---

# 77. Common Mapping Mistake

Bad:

```text
Use EnumType.ORDINAL
```

because:

```text
enum order changes
```

can change the meaning of stored values.

Prefer:

```text
EnumType.STRING
```

for most business enums.

---

# 78. Common Mapping Mistake

Bad:

```text
Application checks uniqueness
```

without:

```text
Database UNIQUE constraint
```

Concurrency can still create duplicates.

---

# 79. Common Mapping Mistake

Bad:

```text
Soft delete without global query strategy
```

A single forgotten query can expose deleted records.

---

# 80. Common Mapping Mistake

Bad:

```text
Store frequently searched relational data inside JSON
```

This can make:

```text
Filtering
Indexing
Constraints
Joins
```

more difficult.

---

# 81. Interview Question

### SINGLE_TABLE vs JOINED?

Answer:

> "SINGLE_TABLE keeps the hierarchy in one table and generally gives simpler polymorphic reads, but can create many nullable columns. JOINED normalizes subtype-specific data into separate tables, but polymorphic queries require more joins."

---

# 82. Interview Question

### What is @MappedSuperclass?

Answer:

> "`@MappedSuperclass` lets entities inherit mapped fields such as IDs and audit fields without making the superclass itself an entity in an inheritance hierarchy."

---

# 83. Interview Question

### What is an embeddable?

Answer:

> "An embeddable represents a value object whose fields are stored as part of the owning entity's table. It's useful for concepts such as Address or Money that don't need independent identity."

---

# 84. Interview Question

### @EmbeddedId vs @IdClass?

Answer:

> "`@EmbeddedId` represents the composite identifier as an embeddable object, while `@IdClass` keeps the identifier fields directly on the entity. Both are standard JPA approaches."

---

# 85. Interview Question

### Why use @MapsId?

Answer:

> "`@MapsId` is useful when an entity relationship contributes to the entity's identifier. It helps align the association mapping with the identifier without unnecessarily duplicating key state."

---

# 86. Interview Question

### Why prefer EnumType.STRING?

Answer:

> "ORDINAL stores numeric positions, so changing enum order can change the meaning of existing database values. STRING stores the enum name, which is generally safer and more readable for persisted business state."

---

# 87. Interview Question

### What is AttributeConverter?

Answer:

> "AttributeConverter converts between a Java attribute representation and its database representation. It's useful for legacy formats, custom enum values, flags and value-object conversions."

---

# 88. Interview Question

### What is soft delete?

Answer:

> "Soft delete marks a record as deleted rather than physically removing it. It can support recovery and auditing, but every relevant query and uniqueness rule must account for deleted records."

---

# 89. Interview Question

### Why are database constraints important if validation exists?

Answer:

> "Application validation improves user experience, but only the database can reliably enforce constraints under concurrent writes. For example, a UNIQUE constraint prevents two concurrent requests from creating the same supposedly unique value."

---

# 90. Interview Scenario

### "You need Card, UPI and Bank Transfer payments."

Answer approach:

```text
Evaluate inheritance
 ↓
Check common fields
 ↓
Check subtype-specific fields
 ↓
Check polymorphic query frequency
 ↓
Choose SINGLE_TABLE or JOINED
```

---

# 91. Interview Scenario

### "OrderItem needs quantity and price."

Don't use a simple many-to-many.

Use:

```text
Order
 ↓
OrderItem
 ↓
Product
```

with:

```text
quantity
unitPrice
discount
```

on `OrderItem`.

---

# 92. Interview Scenario

### "Product has flexible metadata."

Possible:

```text
JSON column
```

if:

```text
Schema is genuinely flexible
```

But use relational columns for frequently queried/strongly constrained fields.

---

# 93. Interview Scenario

### "Two users create the same email concurrently."

Solution:

```text
Application validation
+
Database UNIQUE constraint
```

The database is the final integrity boundary.

---

# 94. Interview Scenario

### "A deleted product should remain available for old orders."

Soft delete can work:

```text
Product
deletedAt = timestamp
```

while:

```text
OrderItem
stores historical product information needed by the business
```

Don't rely on the current product record alone for historical order truth.

---

# 95. Production Checklist

```text
□ Choose inheritance intentionally
□ Use @MappedSuperclass for shared mappings
□ Use embeddables for value objects
□ Model association entities when relationships have data
□ Use stable enum persistence
□ Add database constraints
□ Design soft delete carefully
□ Consider audit requirements
□ Use converters for appropriate representations
□ Evaluate JSON vs relational columns
□ Consider UUID trade-offs
□ Protect tenant isolation
□ Keep historical business data as snapshots where required
```

---

# 96. Advanced Mapping Mental Model

Think:

```text
Java Model
    ↓
Entity or Value Object?
    ↓
Identity?
    ↓
Lifecycle?
    ↓
Relationship?
    ↓
Database representation
    ↓
Constraints
    ↓
Query patterns
    ↓
Performance
```

---

# 97. Real-World E-commerce Mapping

A practical design might look like:

```text
User
 ├── Address (Embeddable / Entity depending on lifecycle)
 │
 └── Order
      ├── OrderItem
      │     └── Product
      │
      └── Payment
             ├── CardPayment
             ├── UpiPayment
             └── BankPayment
```

Supporting concepts:

```text
BaseEntity
Audit fields
OrderStatus
PaymentStatus
Soft delete
Database constraints
Indexes
```

---

# 98. Final Interview Answer

If asked:

> "How do you design Hibernate mappings for a real e-commerce system?"

Say:

> "I first distinguish entities from value objects and model relationships based on business lifecycle. For example, OrderItem would be an association entity because it contains quantity and historical pricing. I'd use embeddables for value objects such as Address, a mapped superclass for common audit fields, and explicit enum/string mappings. For inheritance such as payment types, I'd choose the strategy based on query patterns and schema requirements. Finally, I'd enforce important invariants with database constraints and keep large relationships lazy and query-driven."

---

# 99. Revision Checklist

```text
□ Entity inheritance
□ SINGLE_TABLE
□ Discriminator column
□ JOINED
□ TABLE_PER_CLASS
□ Inheritance trade-offs
□ @MappedSuperclass
□ @Embeddable
□ @Embedded
□ @AttributeOverrides
□ Composite keys
□ @EmbeddedId
□ @IdClass
□ @MapsId
□ Auditing
□ @CreatedDate
□ @LastModifiedDate
□ @CreatedBy
□ @LastModifiedBy
□ Entity listeners
□ @PrePersist
□ @PreUpdate
□ Enums
□ EnumType.STRING
□ EnumType.ORDINAL
□ AttributeConverter
□ UUIDs
□ Natural IDs
□ Surrogate keys
□ Database constraints
□ Soft delete
□ Hibernate restrictions/filters
□ Multi-tenancy
□ JSON columns
□ Custom types
□ Association entities
□ Snapshot pricing
□ E-commerce mappings
□ Interview scenarios
```

---

# 100. What Comes Next

```text
File 09 → Hibernate Interview Mastery & Troubleshooting
```

Next we will combine the previous topics into interview-focused preparation:

```text
Most Asked Hibernate Questions
Scenario-Based Questions
N+1 Debugging
LazyInitializationException
Transaction Bugs
Locking Problems
Slow Query Diagnosis
Mapping Problems
Common Production Errors
Spring Data JPA Traps
Hibernate Logs
SQL Debugging
Coding Questions
System Design Connections
E-commerce Scenarios
Project-Based Questions
Rapid Revision
```

The key interview lesson is:

> **Advanced Hibernate is about modeling the domain correctly while understanding the database consequences. A good mapping is not just valid JPA—it should preserve business semantics, database integrity, query performance and maintainability.**
