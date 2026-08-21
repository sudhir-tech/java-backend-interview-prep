# Microservices — Service Decomposition & Bounded Contexts

This file focuses on one of the most important microservices interview topics:

> How do you decide where one microservice should begin and another should end?

Creating services is easy.

Creating **good service boundaries** is the difficult part.

---

# 1. What Is Service Decomposition?

Service decomposition means breaking a larger application into services based on meaningful business responsibilities.

Instead of:

```text
E-commerce Application
```

you might identify:

```text
User
Catalog
Inventory
Order
Payment
Notification
```

The goal is not to create as many services as possible.

The goal is to create **well-defined, independently manageable business capabilities**.

---

# 2. The Most Important Rule

Don't decompose based only on database tables.

Bad approach:

```text
User table
   ↓
User Service

Product table
   ↓
Product Service

Address table
   ↓
Address Service

Phone table
   ↓
Phone Service
```

This can create unnecessary network calls and tightly coupled services.

Better:

```text
Business capability
       ↓
Domain boundary
       ↓
Service
```

---

# 3. What Is a Business Capability?

A business capability is something the business needs to accomplish.

For an e-commerce company:

```text
Manage customer accounts
Manage product catalog
Manage inventory
Process orders
Process payments
Send notifications
```

These are stronger candidates for service boundaries than individual tables.

---

# 4. Example E-Commerce Decomposition

A reasonable starting point might be:

```text
                    E-Commerce
                         |
       +-----------------+------------------+
       |        |        |        |         |
       ↓        ↓        ↓        ↓         ↓
     User   Catalog   Inventory  Order   Payment
                                             |
                                             ↓
                                       Notification
```

The exact decomposition depends on:

```text
Business requirements
Team ownership
Scale
Data ownership
Deployment needs
```

---

# 5. What Is a Bounded Context?

Bounded Context is a concept from Domain-Driven Design (DDD).

It defines a boundary within which:

```text
A domain model
A language
Business rules
Definitions
```

have a specific meaning.

The same word can mean different things in different contexts.

---

# 6. Example: "Customer"

In a shopping context:

```text
Customer
→ person placing orders
```

In a payment context:

```text
Customer
→ entity associated with payment information
```

In support:

```text
Customer
→ person associated with support tickets
```

You don't necessarily need one giant Customer model shared by every service.

---

# 7. Bounded Context and Microservice

A bounded context and a microservice are related but not identical.

Think:

```text
Bounded Context
      ↓
Logical domain boundary
```

and:

```text
Microservice
      ↓
Deployment/runtime boundary
```

A bounded context can often map naturally to a microservice, but the relationship is not strictly one-to-one.

---

# 8. Domain Model

A domain model represents business concepts and rules.

Example:

```text
Order
 ├── OrderItem
 ├── Customer
 ├── OrderStatus
 └── PaymentStatus
```

But different contexts may model the same business concept differently.

That is normal.

---

# 9. Ubiquitous Language

DDD encourages a shared vocabulary between:

```text
Developers
Product managers
Business experts
Architects
```

For example:

```text
"Order"
"Cancelled"
"Shipped"
"Payment"
"Refund"
```

The team should agree on what these terms mean.

---

# 10. Why Language Matters

Suppose the business says:

> "Cancel the order."

Does that mean:

```text
Stop processing?
Refund payment?
Release inventory?
Stop shipment?
```

Different domains may interpret cancellation differently.

Clear domain language helps establish boundaries and contracts.

---

# 11. Strong Boundary Signals

A potential service boundary becomes stronger when:

```text
Business responsibility is distinct
Business rules are different
Data ownership is clear
Team ownership is clear
Deployment can be independent
Scaling requirements differ
Security requirements differ
```

The more of these you have, the stronger the candidate boundary.

---

# 12. Weak Boundary Signals

Be cautious when:

```text
Two services always change together
They constantly call each other
They share the same database tables
They require one transaction
They cannot be deployed independently
They share most business logic
```

These may indicate that the boundary is wrong.

---

# 13. The Distributed Monolith Warning

Suppose:

```text
Order Service
   ↓
Customer Service
   ↓
Address Service
   ↓
Country Service
   ↓
Currency Service
```

Every order request now requires multiple network calls.

You may have technically created microservices, but operationally:

```text
Everything depends on everything.
```

That is a distributed monolith.

---

# 14. Coupling

Coupling means how strongly components depend on each other.

### Tight coupling

```text
A changes
 ↓
B must change
 ↓
C must change
```

### Loose coupling

```text
A changes internally
 ↓
External contract remains stable
 ↓
B continues working
```

Microservices aim for loose coupling.

---

# 15. Temporal Coupling

Temporal coupling means one service requires another service to be available at exactly the same time.

Example:

```text
Order request
 ↓
Order Service
 ↓
Payment Service
```

If Payment is down:

```text
Order cannot proceed
```

This may be acceptable for critical payment processing.

But for:

```text
Email notification
Analytics
Audit enrichment
```

asynchronous processing may remove unnecessary temporal coupling.

---

# 16. Data Coupling

Bad:

```text
Order Service
       ↓
Shared Orders Table
       ↑
Reporting Service
```

Now both services depend on the same schema.

Better:

```text
Order Service
 ↓
Order DB
 ↓
OrderCreated event
 ↓
Reporting Service
 ↓
Reporting DB
```

The reporting model can evolve independently.

---

# 17. Shared Database

A shared database can be useful during migration from a monolith.

But long term it can create:

```text
Schema coupling
Ownership confusion
Coordinated migrations
Cross-service dependencies
```

Database-per-service improves ownership.

---

# 18. Database Ownership

If Order Service owns:

```text
orders
order_items
```

then another service should not directly update them.

Instead:

```text
Order Service API
```

or:

```text
Order events
```

should be used.

This keeps ownership clear.

---

# 19. Service APIs as Contracts

A service should expose a contract rather than its internal database.

Example:

```text
GET /orders/{id}
```

The consumer should not care whether Order Service stores data in:

```text
MySQL
PostgreSQL
MongoDB
```

The internal implementation can change without breaking consumers.

---

# 20. Don't Share Domain Entities

Avoid having:

```text
common-domain.jar
```

containing every entity:

```text
User
Order
Payment
Product
Inventory
```

and importing it into every microservice.

This can create hidden coupling.

A service should own its own domain model.

---

# 21. Shared Libraries: Use Carefully

Sharing technical libraries can be reasonable:

```text
Logging utility
Tracing utility
Security helper
Common API error format
```

But sharing business-domain logic can create coupling.

Bad:

```text
Order rules library
Payment rules library
Inventory rules library
```

used by every service.

---

# 22. DRY vs Service Independence

In a monolith:

```text
Don't Repeat Yourself
```

is often straightforward.

In microservices:

```text
Some duplication can be healthier
than tightly coupling services.
```

Example:

```text
UserSummaryDTO
```

may legitimately exist in multiple services.

Independent ownership can matter more than eliminating every duplicated class.

---

# 23. Example: Customer Data

User Service owns:

```text
User
Email
Phone
Profile
```

Order Service might store:

```text
customerId
customerNameSnapshot
```

Why a snapshot?

Because order history may need to preserve the name/address used at purchase time.

You don't always need to call User Service every time you display an old order.

---

# 24. Snapshot vs Live Data

Consider:

```text
Customer name:
"John"

Order placed

Customer changes name:
"John Smith"
```

Should the old order display:

```text
John
```

or:

```text
John Smith
```

Business requirements determine this.

If historical accuracy matters:

```text
Order stores a snapshot
```

This can reduce runtime coupling.

---

# 25. Service Boundary Based on Change

A useful question:

> What parts of the system tend to change together?

If:

```text
A
B
C
```

always change together, separating them may create unnecessary deployment coupling.

If:

```text
Payment
```

changes independently from:

```text
Catalog
```

that supports separation.

---

# 26. Service Boundary Based on Scale

Suppose:

```text
Product reads = 100,000/sec
Order writes = 1,000/sec
```

Separating them allows:

```text
Product Service
→ many instances/cache

Order Service
→ fewer instances
```

Independent scaling becomes valuable.

---

# 27. Service Boundary Based on Security

Payment processing may have very different security requirements.

For example:

```text
Payment Service
```

may need:

```text
Restricted access
Audit logging
Stronger secrets management
Special compliance controls
```

This can be a strong reason for isolation.

---

# 28. Service Boundary Based on Team Ownership

Suppose:

```text
Catalog Team
Order Team
Payment Team
```

Each team owns a domain.

A natural architecture might be:

```text
Catalog → Catalog Service
Order → Order Service
Payment → Payment Service
```

This reduces coordination.

---

# 29. Conway's Law

Conway's Law suggests that system structure tends to mirror organizational communication structures.

If three teams communicate through independent channels, architecture may naturally evolve toward:

```text
Team A → Service A
Team B → Service B
Team C → Service C
```

This is why team ownership matters in microservice design.

---

# 30. Domain-Driven Design

DDD provides concepts useful for service decomposition:

```text
Domain
Subdomain
Bounded Context
Entity
Value Object
Aggregate
Domain Event
Ubiquitous Language
```

You don't need to use every DDD pattern to build microservices.

---

# 31. Core Domain

The core domain is the part that provides the company's major competitive/business value.

Example:

For an e-commerce company:

```text
Recommendation engine
Pricing
Order fulfillment
Marketplace matching
```

depending on the business.

Core domains usually deserve the strongest modeling and engineering attention.

---

# 32. Supporting Domain

Supporting domains are important but may not be the company's main differentiator.

Examples:

```text
Customer management
Internal workflows
Reporting
```

depending on the company.

---

# 33. Generic Domain

Generic capabilities can often be implemented using existing solutions.

Examples:

```text
Authentication
Email delivery
Logging
Payments
```

depending on the business and context.

Be careful: payment may actually be a core domain for a fintech company.

Domain classification is business-specific.

---

# 34. Subdomain

A subdomain is a distinct area of the business domain.

Example:

```text
E-commerce
│
├── Catalog
├── Inventory
├── Ordering
├── Payments
└── Delivery
```

These can become candidates for bounded contexts.

---

# 35. Aggregate

An aggregate is a consistency boundary in DDD.

Example:

```text
Order
 ├── OrderItem
 └── ShippingAddress
```

The Order may be the aggregate root.

External code should interact with the aggregate through its root rather than directly modifying internal objects arbitrarily.

---

# 36. Aggregate Root

Example:

```text
Order
```

can be the aggregate root for:

```text
OrderItem
ShippingAddress
```

Conceptually:

```text
Order
  ↓
controls changes
  ↓
OrderItem
```

This helps define transaction boundaries.

---

# 37. Why Aggregates Matter in Microservices

Aggregates help answer:

```text
What should be strongly consistent?
```

If:

```text
Order
OrderItem
```

must change together, they may belong to the same aggregate/transaction boundary.

If:

```text
Order
Notification
```

doesn't need immediate consistency, notification can be asynchronous.

---

# 38. Aggregate Size

Avoid huge aggregates.

Bad:

```text
Order
 ↓
Customer
 ↓
All Customer Orders
 ↓
Payment
 ↓
Inventory
```

This creates large consistency boundaries.

Prefer smaller aggregates with explicit interactions.

---

# 39. Entity vs Value Object

Entity:

```text
Has identity
```

Example:

```text
Order ID = 123
```

Value object:

```text
Defined by its value
```

Example:

```text
Money
Address
Coordinates
```

Value objects can help make domain models clearer.

---

# 40. Domain Event

A domain event represents something that happened.

Examples:

```text
OrderPlaced
PaymentCompleted
InventoryReserved
OrderCancelled
```

Events can be consumed by other services.

---

# 41. Command vs Event

Command:

```text
"Do this."
```

Example:

```text
ProcessPayment
```

Event:

```text
"This happened."
```

Example:

```text
PaymentCompleted
```

Commands usually express intent.

Events express completed facts.

---

# 42. Event Ownership

A service should generally publish events about changes in the domain it owns.

Example:

```text
Order Service
   ↓
OrderCreated
```

Payment Service should not publish:

```text
OrderCreated
```

because it doesn't own orders.

It may publish:

```text
PaymentCompleted
```

---

# 43. Integration Events

An integration event is an event intended for communication between bounded contexts/services.

Example:

```text
Order Service
     ↓
OrderConfirmed
     ↓
Kafka
     ↓
Notification Service
```

The event should contain enough information for the consumer without exposing internal database details unnecessarily.

---

# 44. Avoid Chatty Services

Bad:

```text
Order → User
Order → Address
Order → Product
Order → Inventory
Order → Pricing
Order → Tax
```

for every request.

This creates:

```text
High latency
More failure points
More network traffic
Tight runtime coupling
```

Look for ways to:

```text
Cache
Snapshot
Batch
Use events
Create appropriate read models
```

when business requirements allow.

---

# 45. Orchestrating Too Much

A giant orchestrator can become:

```text
God Service
```

Example:

```text
Order Orchestrator
 ↓
User
 ↓
Product
 ↓
Inventory
 ↓
Payment
 ↓
Shipping
 ↓
Notification
```

If every business rule lives in the orchestrator, individual services lose meaningful ownership.

Use orchestration where it genuinely simplifies a business workflow.

---

# 46. Circular Dependencies

Bad:

```text
Order → Payment
Payment → Order
```

This creates difficult runtime and architectural coupling.

Prefer clear dependency direction or asynchronous events where appropriate.

---

# 47. Dependency Graph

Imagine:

```text
Order
 ├── Payment
 ├── Inventory
 └── User
```

Ask:

```text
Which dependencies are required?
Which can be asynchronous?
Which can be cached?
Which can be represented as local data?
```

The goal is not zero dependencies.

The goal is **manageable dependencies**.

---

# 48. Strong vs Weak Coupling

Strong coupling:

```text
Order cannot function
unless User API responds.
```

Weak coupling:

```text
Order has required customer data locally
and receives user updates asynchronously.
```

The second can be more resilient but introduces eventual consistency.

This is a trade-off.

---

# 49. Data Duplication in Microservices

Duplication is not automatically bad.

Example:

```text
User Service
User profile

Order Service
Customer name snapshot
```

The duplication can be intentional.

The key question is:

```text
Who owns the source of truth?
```

---

# 50. Source of Truth

For example:

```text
User Service
     ↓
Source of truth for email
```

Other services may have cached copies.

But they should not independently modify the canonical user email.

---

# 51. Read Models

Suppose the UI needs:

```text
Order
Customer name
Product name
Payment status
Shipping status
```

Calling five services for every screen can be expensive.

A dedicated read model can combine data asynchronously:

```text
Events
 ↓
Read Model
 ↓
Order Dashboard API
```

This is a common CQRS-style approach.

---

# 52. CQRS

CQRS means:

```text
Command Query Responsibility Segregation
```

It separates:

```text
Write model
```

from:

```text
Read model
```

They can use different models/storage optimized for their jobs.

CQRS is optional—not a requirement for microservices.

---

# 53. Example CQRS

```text
Commands
   ↓
Order Service
   ↓
Order DB

Events
   ↓
Read Model
   ↓
Order Dashboard
```

Writes remain transactional while reads can be optimized for the UI.

---

# 54. Eventual Consistency Trade-Off

If the read model updates asynchronously:

```text
Order created
 ↓
Event published
 ↓
Read model updates later
```

There may be a short period where:

```text
Order exists
but dashboard doesn't show it yet.
```

Business requirements must decide whether this is acceptable.

---

# 55. How to Choose Boundaries

A practical checklist:

```text
1. What business capability does this represent?
2. What business rules belong here?
3. Who owns the data?
4. Which team owns it?
5. What changes together?
6. What scales differently?
7. What needs strong consistency?
8. What can be asynchronous?
9. What security boundary exists?
10. Can it be deployed independently?
```

---

# 56. Example: Splitting an E-Commerce Monolith

Start:

```text
Ecommerce Application
```

Identify domains:

```text
Catalog
Inventory
Orders
Payments
Notifications
Users
```

Then evaluate:

```text
Catalog
→ high read volume
→ independent scaling
→ clear ownership

Payment
→ strong security boundary
→ independent rules

Notification
→ asynchronous
→ independent processing
```

These are strong candidates.

---

# 57. Extract Gradually

Don't rewrite the whole monolith in one massive project.

A safer approach:

```text
Monolith
   ↓
Identify boundary
   ↓
Extract one capability
   ↓
Run independently
   ↓
Monitor
   ↓
Repeat
```

This is often called the Strangler Fig pattern when functionality is gradually moved behind a new architecture.

---

# 58. Strangler Fig Pattern

Conceptually:

```text
                Client
                   |
                   ↓
              Entry Layer
               /       \
              ↓         ↓
        New Service   Monolith
              |
        gradually grows
              |
        Monolith shrinks
```

Over time:

```text
Monolith → smaller
Microservices → larger
```

---

# 59. Anti-Corruption Layer

When a new service must interact with a legacy system, an anti-corruption layer can translate between models.

Example:

```text
New Order Service
       |
       ↓
Anti-Corruption Layer
       |
       ↓
Legacy Order System
```

This prevents legacy concepts from leaking directly into the new domain model.

---

# 60. Legacy Extraction Example

Legacy system says:

```text
CUSTOMER_TYPE = "C1"
```

New service wants:

```text
CustomerSegment.PREMIUM
```

The translation layer handles:

```text
C1 → PREMIUM
```

The new service doesn't need to understand every legacy convention.

---

# 61. Shared Kernel

DDD has the concept of a shared kernel:

```text
Small set of shared domain concepts
```

Use sparingly.

A large shared kernel can recreate the coupling microservices were supposed to reduce.

---

# 62. Service Size

Don't ask:

> "How many classes should a microservice contain?"

Instead ask:

```text
Does it have a coherent responsibility?
Can its team own it?
Can it evolve independently?
Is its data boundary clear?
```

There is no universal line-count or class-count rule.

---

# 63. One Service = One Database?

The stronger principle is:

```text
One service owns its data.
```

The physical database arrangement can vary:

```text
Separate database
Separate schema
Separate logical ownership
```

depending on infrastructure.

---

# 64. Should Services Share Tables?

Generally:

```text
Avoid direct shared-table ownership.
```

If two services need the same information:

```text
API
or
Events
or
Local read model
```

should usually be considered.

---

# 65. What If Two Services Need the Same Data?

Options:

```text
1. Synchronous API call
2. Event-driven replication
3. Local cache
4. Snapshot
5. Dedicated read model
```

Choose based on:

```text
Freshness
Latency
Availability
Complexity
Consistency
```

---

# 66. Boundary Decision Example

Requirement:

```text
Order page needs customer name.
```

Option A:

```text
Every request
Order → User Service
```

Option B:

```text
Order stores customer name snapshot
```

Option C:

```text
Order read model consumes UserUpdated events
```

There isn't one universal answer.

The correct answer depends on:

```text
Freshness requirement
Traffic
Consistency
Business semantics
```

---

# 67. Interview Scenario

### "How would you decide whether Payment should be a separate service?"

Strong answer:

> I'd look at whether payment has distinct business rules, security/compliance requirements, team ownership, scaling needs and independent deployment requirements. Payment is also a good isolation boundary because failures and sensitive data should be contained. I wouldn't split it purely because there is a payment table.

---

# 68. Interview Scenario

### "Should Inventory and Product be separate services?"

Answer:

> It depends. Catalog data is often read-heavy, while inventory has concurrency-sensitive updates. If their scaling, ownership and consistency requirements differ significantly, separating them can make sense. But I'd avoid splitting them if it would create excessive synchronous calls and operational complexity.

---

# 69. Interview Scenario

### "Should Cart be its own service?"

Possible answer:

> It can be, especially if cart state has different lifecycle, scaling or storage requirements. But for a smaller system, keeping cart inside an Order or Commerce module may be simpler. I'd base the decision on business boundaries and actual operational needs.

---

# 70. Interview Scenario

### "Why not make every bounded context a microservice?"

Answer:

> A bounded context is a logical boundary, while a microservice is a deployment boundary. Some contexts may be better kept together initially, especially when the operational cost of separating them doesn't provide enough value.

---

# 71. Interview Scenario

### "What is the biggest mistake when decomposing a monolith?"

Strong answer:

> Splitting based only on technical layers or database tables. Good decomposition should follow business capabilities and ownership. Otherwise we can create many services that are still tightly coupled and effectively become a distributed monolith.

---

# 72. Interview Scenario

### "How do you reduce service-to-service calls?"

Answer:

> I'd first identify whether each call is truly required. Depending on consistency requirements, I might use local snapshots, caching, event-driven replication, batching or a dedicated read model.

---

# 73. Interview Scenario

### "Is data duplication bad?"

Answer:

> Not necessarily. In microservices, controlled duplication can reduce runtime coupling and improve read performance. The important thing is to define the source of truth and how replicas are updated.

---

# 74. Interview Scenario

### "Why not share entities between services?"

Answer:

> Shared domain entities create compile-time and conceptual coupling. If one service changes the shared model, other services may be forced to change. I'd prefer each service to own its domain model and communicate through stable contracts.

---

# 75. Interview Scenario

### "What is a distributed monolith?"

Answer:

> It's a system where services are deployed separately but remain tightly coupled through shared databases, synchronous dependencies and coordinated releases. It has distributed-system complexity without enough independent-service benefits.

---

# 76. Interview Scenario

### "How would you migrate a monolith?"

Answer:

> I'd identify a bounded context with a clear boundary, extract it incrementally, establish an API or event contract, move ownership of its data, monitor the new service and gradually route traffic to it. I'd avoid rewriting the entire system at once.

---

# 77. Common Decomposition Mistakes

```text
❌ One service per database table
❌ One service per entity
❌ Too many tiny services
❌ Shared database ownership
❌ Shared domain model
❌ Circular dependencies
❌ Excessive synchronous calls
❌ Giant orchestration service
❌ No clear data owner
❌ Services that always deploy together
```

---

# 78. Good Decomposition Characteristics

```text
✓ Business capability
✓ Clear data ownership
✓ Clear team ownership
✓ Independent deployment
✓ Independent scaling where useful
✓ Stable API/event contract
✓ Manageable dependencies
✓ Appropriate consistency boundary
✓ Fault isolation
✓ Security boundary where useful
```

---

# 79. Practical E-Commerce Boundary

One possible design:

```text
USER
├── Profile
├── Authentication
└── Preferences

CATALOG
├── Products
├── Categories
└── Product metadata

INVENTORY
├── Stock
├── Reservation
└── Availability

ORDER
├── Orders
├── Order items
└── Order lifecycle

PAYMENT
├── Payment
├── Refund
└── Payment status

NOTIFICATION
├── Email
├── SMS
└── Push
```

This is only an example.

Real boundaries depend on the business.

---

# 80. Final Mental Model

When decomposing a system:

```text
Business
   ↓
Capabilities
   ↓
Bounded Contexts
   ↓
Data Ownership
   ↓
Service Boundaries
   ↓
Communication Contracts
   ↓
Deployment Boundaries
```

Not:

```text
Tables
 ↓
Services
```

---

# 81. Interview Cheat Sheet

```text
Business capability
→ What the business does

Bounded context
→ Where a domain model and language have a defined meaning

Service boundary
→ Where independent responsibility/ownership exists

Data ownership
→ Which service controls a piece of data

Aggregate
→ Consistency boundary

Domain event
→ Something meaningful that happened

Command
→ Request to perform an action

Loose coupling
→ Services can evolve independently

Temporal coupling
→ Services must be available together

Distributed monolith
→ Separate deployments but tight runtime/deployment coupling

Strangler pattern
→ Gradually replace/extract functionality from a monolith

Anti-corruption layer
→ Translate legacy/external models into the new domain

Read model
→ Data optimized for a particular read use case
```

---

# 82. Final Interview Answer

If asked:

> "How would you decide microservice boundaries?"

Use this answer:

> "I'd start with business capabilities and domain boundaries rather than database tables. I'd identify bounded contexts, clarify data ownership and business rules, and look at which components change, scale and deploy independently. I'd also consider team ownership, security and consistency requirements. Finally, I'd evaluate runtime dependencies because if two proposed services constantly call each other or must always deploy together, the boundary may not be right. The goal is loose coupling with clear ownership, not simply creating more services."

That is the core skill behind effective microservice decomposition.
