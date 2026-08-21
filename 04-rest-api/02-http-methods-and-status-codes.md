# HTTP Methods and Status Codes

This file focuses on HTTP semantics that every Java/Spring Boot backend developer should understand when designing REST APIs.

---

# 1. HTTP Methods

The most common methods used in REST APIs are:

```text
GET
POST
PUT
PATCH
DELETE
```

The method tells the server the intended operation.

---

# 2. GET

GET is used to retrieve a resource or collection.

Example:

```http
GET /api/products/100
```

Response:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 75000
}
```

GET should not intentionally modify server state.

### Spring Boot

```java
@GetMapping("/{id}")
public ProductResponse getProduct(
        @PathVariable Long id) {

    return productService.getProduct(id);
}
```

---

# 3. GET Collection

To retrieve multiple resources:

```http
GET /api/products
```

For large collections, use pagination:

```text
GET /api/products?page=0&size=20
```

Filtering:

```text
GET /api/products?category=phones
```

Sorting:

```text
GET /api/products?sort=price,desc
```

---

# 4. POST

POST is commonly used to create a new resource under a collection.

Example:

```http
POST /api/products
Content-Type: application/json
```

Request:

```json
{
  "name": "Laptop",
  "price": 75000
}
```

Response:

```http
201 Created
Location: /api/products/100
```

POST is generally **not idempotent**.

If the same request is submitted twice, it may create two resources.

---

# 5. PUT

PUT targets a specific resource and is generally used to replace its representation.

Example:

```http
PUT /api/products/100
```

Request:

```json
{
  "name": "Laptop Pro",
  "price": 85000
}
```

PUT is generally **idempotent**.

Sending the same PUT request multiple times should result in the same intended resource state.

---

# 6. PUT Example

Suppose the current resource is:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 75000
}
```

Request:

```http
PUT /api/products/100
```

```json
{
  "name": "Laptop Pro",
  "price": 85000
}
```

The intended new representation is:

```json
{
  "id": 100,
  "name": "Laptop Pro",
  "price": 85000
}
```

If the API treats PUT as replacement, clients should provide the complete representation expected by the endpoint.

---

# 7. PATCH

PATCH is used for partial modifications.

Example:

```http
PATCH /api/products/100
Content-Type: application/json
```

Request:

```json
{
  "price": 80000
}
```

Only the specified field is modified.

PATCH idempotency depends on the specific patch operation.

---

# 8. DELETE

DELETE requests removal of a resource.

Example:

```http
DELETE /api/products/100
```

A successful operation may return:

```http
204 No Content
```

Spring Boot:

```java
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteProduct(
        @PathVariable Long id) {

    productService.delete(id);
}
```

DELETE is generally idempotent.

Repeated DELETE requests should have the same intended effect on the resource state, although the HTTP response itself may differ.

---

# 9. HEAD

HEAD is similar to GET but the server does not return the response body.

It can be useful for checking:

```text
Whether a resource exists
Response headers
Content length
Cache information
```

Example:

```http
HEAD /api/products/100
```

---

# 10. OPTIONS

OPTIONS asks what communication options are available for a resource.

It is commonly encountered with CORS and browser preflight requests.

Example:

```http
OPTIONS /api/products
```

A response may indicate allowed methods:

```http
Allow: GET, POST, PUT, DELETE
```

---

# 11. Safe Methods

A safe HTTP method does not intentionally modify server state.

Common safe methods:

```text
GET
HEAD
OPTIONS
```

Safe does not mean:

```text
No database access
No logging
No server-side work
```

It means the method's intended semantics are read-oriented rather than state-changing.

---

# 12. Idempotent Methods

An idempotent method has the same intended effect when the same request is repeated.

Common idempotent methods:

```text
GET
HEAD
OPTIONS
PUT
DELETE
```

POST is generally not idempotent.

PATCH depends on the operation.

Important:

> Idempotent does not mean every repeated response must be identical.

---

# 13. POST vs PUT

A common interview question.

### POST

```text
POST /products
```

The server typically chooses the new resource identifier.

### PUT

```text
PUT /products/100
```

The client targets a specific resource URI.

Interview answer:

> POST is commonly used to create a resource under a collection and is generally not idempotent. PUT targets a specific resource and is generally idempotent, commonly replacing its representation.

---

# 14. PUT vs PATCH

### PUT

```text
PUT /products/100
```

Generally represents replacement of the target resource.

### PATCH

```text
PATCH /products/100
```

Represents partial modification.

Interview answer:

> PUT is generally used for replacing a resource representation and is idempotent. PATCH is used for partial modifications, and its idempotency depends on the patch operation.

---

# 15. HTTP Status Code Categories

HTTP status codes have five main classes:

```text
1xx → Informational
2xx → Success
3xx → Redirection
4xx → Client Error
5xx → Server Error
```

The most important ones for backend interviews are:

```text
200
201
204
400
401
403
404
405
409
415
422
429
500
502
503
504
```

---

# 16. 200 OK

Means the request succeeded.

Example:

```http
GET /api/products/100
```

Response:

```http
200 OK
```

with a response body.

Typical use:

```text
GET successful
PUT successful with representation returned
PATCH successful with representation returned
```

---

# 17. 201 Created

Indicates that a new resource was successfully created.

Example:

```http
POST /api/products
```

Response:

```http
201 Created
Location: /api/products/100
```

A response body may also contain the created representation.

Interview point:

> 201 is more precise than returning 200 for successful resource creation.

---

# 18. 204 No Content

Means the operation succeeded but there is no response body.

Common example:

```http
DELETE /api/products/100
```

Response:

```http
204 No Content
```

It can also be used for successful updates where the API intentionally returns no representation.

---

# 19. 400 Bad Request

Indicates that the request is invalid.

Examples:

```text
Malformed JSON
Invalid parameter syntax
Invalid request format
```

Example:

```json
{
  "price": "abc"
}
```

when a numeric price is required.

---

# 20. 401 Unauthorized

Generally means the request lacks valid authentication credentials.

Examples:

```text
Missing access token
Invalid token
Expired token
```

Example:

```http
GET /api/orders
Authorization: Bearer invalid-token
```

Response:

```http
401 Unauthorized
```

Interview shortcut:

```text
401 → Who are you?
```

---

# 21. 403 Forbidden

Means the server understood the request, but the authenticated client is not permitted to perform it.

Example:

```text
Authenticated user
        ↓
POST /api/admin/products
        ↓
User is not an admin
        ↓
403 Forbidden
```

Interview shortcut:

```text
403 → I know who you are, but you can't do this.
```

---

# 22. 401 vs 403

This is one of the most common Spring Security interview questions.

```text
401 → Authentication failure
403 → Authorization failure
```

Example:

```text
No valid JWT
    → 401

Valid JWT + insufficient role
    → 403
```

---

# 23. 404 Not Found

Means the requested resource could not be found.

Example:

```http
GET /api/products/999999
```

Response:

```http
404 Not Found
```

Spring Boot example:

```java
throw new ResourceNotFoundException(
    "Product not found"
);
```

The global exception handler can convert this to 404.

---

# 24. 405 Method Not Allowed

Means the server knows the resource, but the HTTP method is not supported for that endpoint.

Example:

```http
PATCH /api/products
```

when the endpoint only supports:

```text
GET
POST
```

The response can include:

```http
Allow: GET, POST
```

---

# 25. 409 Conflict

Indicates a conflict with the current state of the resource.

Examples:

```text
Duplicate username
Duplicate product SKU
Optimistic locking conflict
Order state conflict
```

Example:

```text
POST /users
email = existing@example.com
```

The API may return:

```http
409 Conflict
```

---

# 26. 415 Unsupported Media Type

Means the server does not support the media type sent in the request.

Example:

```http
Content-Type: application/xml
```

when the API only accepts:

```http
Content-Type: application/json
```

Response:

```http
415 Unsupported Media Type
```

---

# 27. 422 Unprocessable Content

Can be used when the request format is understood but the submitted data fails semantic validation.

Example:

```json
{
  "age": -10
}
```

The JSON is valid, but the value is not acceptable according to the application's rules.

Some APIs use 400 for validation failures instead. The important thing is to define a consistent API convention.

---

# 28. 429 Too Many Requests

Indicates that the client has sent too many requests in a given period.

Example:

```text
100 requests/minute
```

If exceeded:

```http
429 Too Many Requests
```

A server may include:

```http
Retry-After: 60
```

This is common with rate limiting.

---

# 29. 500 Internal Server Error

Indicates an unexpected server-side failure.

Example causes:

```text
Unhandled exception
Unexpected application failure
Unexpected database failure
```

Do not return:

```text
Stack trace
SQL credentials
Internal paths
Secret values
```

Instead:

```text
Client → controlled error
Server → detailed logs
```

---

# 30. 502 Bad Gateway

Usually means a gateway or proxy received an invalid response from an upstream server.

Example:

```text
Client
  ↓
API Gateway
  ↓
Order Service
  X
```

The gateway may return:

```http
502 Bad Gateway
```

---

# 31. 503 Service Unavailable

Means the service is temporarily unable to handle the request.

Possible causes:

```text
Service overloaded
Service temporarily down
Maintenance
Dependency unavailable
```

A retry may be appropriate depending on the operation.

---

# 32. 504 Gateway Timeout

Means a gateway or proxy did not receive a timely response from an upstream service.

Example:

```text
Client
  ↓
Gateway
  ↓
Payment Service
  ↓
Timeout
```

The gateway can return:

```http
504 Gateway Timeout
```

---

# 33. Status Code Decision Tree

A simple mental model:

```text
Request
   |
   ├── Authentication missing/invalid?
   |       └── 401
   |
   ├── Authenticated but not allowed?
   |       └── 403
   |
   ├── Invalid request?
   |       └── 400 / 422
   |
   ├── Resource missing?
   |       └── 404
   |
   ├── Method unsupported?
   |       └── 405
   |
   ├── State conflict?
   |       └── 409
   |
   ├── Rate limit exceeded?
   |       └── 429
   |
   ├── Server failure?
   |       └── 500
   |
   └── Upstream/gateway issue?
           └── 502 / 503 / 504
```

---

# 34. HTTP Headers and Status Codes

Headers provide metadata around the request and response.

Example:

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/products/100
Cache-Control: no-cache
```

The status code communicates the outcome while headers provide additional information.

---

# 35. Location Header

The `Location` header identifies a newly created resource.

Example:

```http
POST /api/products
```

Response:

```http
201 Created
Location: /api/products/100
```

This is especially useful for POST requests that create resources.

---

# 36. Allow Header

The `Allow` header can communicate which methods are supported.

Example:

```http
Allow: GET, POST
```

It is especially relevant to `405 Method Not Allowed`.

---

# 37. Retry-After

Can tell clients when to retry.

Example:

```http
429 Too Many Requests
Retry-After: 60
```

It can also be used with some temporary service-unavailable responses.

---

# 38. Cache-Control

Controls caching behavior.

Example:

```http
Cache-Control: max-age=300
```

This can tell caches that a response may be considered fresh for 300 seconds.

For sensitive or non-cacheable data, applications should choose appropriate directives.

---

# 39. ETag

An ETag identifies a particular representation version.

Example:

```http
ETag: "product-100-v7"
```

A client can later send:

```http
If-None-Match: "product-100-v7"
```

If the resource has not changed, the server may respond:

```http
304 Not Modified
```

This can reduce unnecessary response-body transfer.

---

# 40. Conditional Requests

Conditional headers can also protect against lost updates.

Example:

```http
If-Match: "product-100-v7"
```

The server updates the resource only if the representation still matches that version.

If another client changed it first, the API can return a conflict-related response such as:

```text
412 Precondition Failed
```

or another status according to the API's concurrency design.

---

# 41. 304 Not Modified

`304 Not Modified` is used with conditional GET requests when the cached representation is still valid.

Example:

```http
GET /api/products/100
If-None-Match: "product-100-v7"
```

If unchanged:

```http
304 Not Modified
```

The server does not need to resend the full representation.

---

# 42. 412 Precondition Failed

Used when a request condition supplied by the client is not satisfied.

Example:

```http
PUT /api/products/100
If-Match: "old-version"
```

If the resource changed:

```http
412 Precondition Failed
```

This is useful for optimistic concurrency control.

---

# 43. Idempotency and Distributed Systems

Imagine a payment request:

```http
POST /api/payments
```

The network fails after the server processes the payment.

The client may retry.

Without an idempotency mechanism:

```text
Request 1 → Payment created
Network failure
Request 2 → Another payment created
```

A client can send an idempotency key:

```http
Idempotency-Key: 8f42a1
```

The server can recognize the retry and avoid creating a duplicate operation.

This is especially useful for:

```text
Payments
Orders
Bookings
External API calls
```

---

# 44. HTTP Method + Status Code Examples

## Create

```http
POST /api/products
```

```text
201 Created
```

## Read

```http
GET /api/products/100
```

```text
200 OK
```

## Update

```http
PUT /api/products/100
```

```text
200 OK
```

or:

```text
204 No Content
```

## Partial update

```http
PATCH /api/products/100
```

```text
200 OK
```

or:

```text
204 No Content
```

## Delete

```http
DELETE /api/products/100
```

```text
204 No Content
```

---

# 45. Spring Boot Example

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> getProduct(
            @PathVariable Long id) {

        return ResponseEntity.ok(
            productService.getProduct(id)
        );
    }

    @PostMapping
    public ResponseEntity<ProductResponse> createProduct(
            @Valid @RequestBody CreateProductRequest request) {

        ProductResponse response =
            productService.create(request);

        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(response);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(
            @PathVariable Long id) {

        productService.delete(id);

        return ResponseEntity.noContent().build();
    }
}
```

---

# 46. Common Interview Questions

## What is the difference between GET and POST?

> GET is used to retrieve resources and is safe and idempotent. POST is commonly used to create resources or trigger processing and is generally not idempotent.

## Is POST idempotent?

> HTTP does not define POST as inherently idempotent. Repeating a POST can create multiple resources or operations unless the application implements an idempotency mechanism.

## Is DELETE idempotent?

> DELETE is generally idempotent because repeating it should have the same intended effect on the resource state. However, the response can differ between the first and later requests.

## Is PATCH idempotent?

> PATCH is not inherently idempotent. It depends on the specific patch operation and how the server implements it.

## PUT vs PATCH?

> PUT generally replaces the resource representation at a specific URI, while PATCH applies a partial modification.

## What is 201?

> 201 Created indicates that the request successfully created a resource. The response can include a Location header identifying the new resource.

## 401 vs 403?

> 401 means valid authentication credentials are missing or invalid. 403 means the client is authenticated but lacks permission.

## 400 vs 422?

> 400 is commonly used for invalid requests, while 422 can be used when the request is syntactically valid but the content fails semantic validation. Teams should use a consistent convention.

## 404 vs 405?

> 404 means the requested resource was not found. 405 means the resource exists or is recognized, but the HTTP method is not allowed for that endpoint.

## 409 vs 422?

> 409 is useful when the request conflicts with the current resource state, while 422 can represent semantically invalid input. The exact boundary should be consistent within the API.

## 502 vs 503 vs 504?

> 502 usually indicates an invalid upstream response, 503 indicates temporary service unavailability, and 504 indicates that a gateway or proxy timed out waiting for an upstream response.

---

# 47. Interview Scenario

### Question

A user clicks "Pay" and receives a timeout. The client retries. How would you prevent duplicate payments?

### Answer

> I would make the payment operation idempotent using an idempotency key. The client sends a unique key with the request, and the server stores the result associated with that key. If the same request is retried, the server returns the existing result instead of creating another payment.

---

# 48. Quick Reference

| Method | Typical Use | Safe | Idempotent |
|---|---|---:|---:|
| GET | Read | Yes | Yes |
| HEAD | Read headers | Yes | Yes |
| OPTIONS | Discover capabilities | Yes | Yes |
| POST | Create/process | No | Generally no |
| PUT | Replace | No | Yes |
| PATCH | Partial update | No | Depends |
| DELETE | Remove | No | Yes |

---

# 49. Status Code Quick Reference

| Code | Meaning | Common Example |
|---|---|---|
| 200 | OK | Successful GET |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 304 | Not Modified | Conditional GET |
| 400 | Bad Request | Invalid request |
| 401 | Unauthorized | Invalid/missing authentication |
| 403 | Forbidden | Insufficient permission |
| 404 | Not Found | Missing resource |
| 405 | Method Not Allowed | Unsupported HTTP method |
| 409 | Conflict | State/unique constraint conflict |
| 412 | Precondition Failed | Failed `If-Match` condition |
| 415 | Unsupported Media Type | Unsupported Content-Type |
| 422 | Unprocessable Content | Semantic validation failure |
| 429 | Too Many Requests | Rate limit |
| 500 | Internal Server Error | Unexpected server failure |
| 502 | Bad Gateway | Invalid upstream response |
| 503 | Service Unavailable | Temporary outage/overload |
| 504 | Gateway Timeout | Upstream timeout |

---

# Final Mental Model

```text
HTTP Method
     ↓
What operation is intended?

Status Code
     ↓
What happened?

Headers
     ↓
What additional metadata/conditions apply?

Body
     ↓
What data is being transferred?
```

For a Java backend interview, remember:

```text
GET    → retrieve
POST   → create/process
PUT    → replace
PATCH  → partially modify
DELETE → remove

200 → success
201 → created
204 → success, no body

400 → bad request
401 → authentication
403 → authorization
404 → not found
405 → method not allowed
409 → conflict
422 → semantic validation
429 → rate limit

500 → server failure
502 → bad upstream response
503 → unavailable
504 → upstream timeout
```

> **Use HTTP methods according to their semantics, choose status codes that accurately describe the result, and design retries carefully—especially for state-changing operations such as payments and orders.**
