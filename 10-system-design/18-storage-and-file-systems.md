# System Design — File 18: Storage & File Systems

Storage is a major part of system design because applications need to store:

- Structured data
- Images
- Videos
- Documents
- Backups
- Logs
- Archives

The right storage choice depends on access pattern, data size, durability, latency, consistency, scalability, and cost.

---

## 1. Main Storage Categories

The three major categories are:

```text
Block Storage
File Storage
Object Storage
```

Other important storage systems include:

```text
Databases
Caches
Search indexes
```

---

## 2. Block Storage

Block storage exposes raw storage blocks to a machine.

```text
Application
    |
    v
File System
    |
    v
Block Storage
```

Common uses:

- Virtual machine disks
- Database disks
- Persistent volumes
- Low-level storage workloads

---

## 3. File Storage

File storage provides files and directories.

```text
/
├── users/
├── reports/
└── uploads/
```

Useful when applications need traditional filesystem semantics or shared directories.

---

## 4. Object Storage

Object storage stores data as objects.

```text
Bucket
  |
  +--> image.jpg
  +--> video.mp4
  +--> report.pdf
```

An object generally contains:

```text
Data
Metadata
Object Key
```

Object storage is commonly used for large unstructured data.

---

## 5. Object Storage Benefits

Good use cases:

- Images
- Videos
- Documents
- Backups
- Logs
- Archives
- Static assets
- Data lakes

Typical advantages:

```text
High durability
Massive scalability
HTTP/API access
Low operational overhead
```

---

## 6. Object Key

An object can have a key such as:

```text
users/123/profile.jpg
```

It may look like a filesystem path, but object storage does not necessarily behave like a traditional hierarchical filesystem.

Think:

```text
Bucket + Key
```

---

## 7. Object Metadata

Metadata can describe:

```text
Content-Type
Size
Creation time
Custom application metadata
Cache directives
```

Example:

```text
Key: products/101/image.jpg
Content-Type: image/jpeg
```

---

## 8. Object Storage vs Database

For many systems, large files should be stored outside the relational database.

Common architecture:

```text
Application
    |
    +--> MySQL
    |     |
    |     +--> Product metadata
    |
    +--> Object Storage
          |
          +--> Product image
```

The database stores the object key/reference and metadata.

Object storage stores the actual file.

---

## 9. Why Not Store Images in MySQL?

Large binary data inside the database can increase:

```text
Database size
Backup size
I/O
Replication traffic
Operational cost
```

For many applications, object storage is a better fit.

This is a design decision, not an absolute rule.

---

## 10. File Upload Architecture

Naive approach:

```text
Client
  |
  v
Spring Boot API
  |
  v
Object Storage
```

For large files, the API may consume significant:

```text
Bandwidth
Memory
Connection capacity
CPU
```

---

## 11. Direct Upload

A scalable approach:

```text
Client
  |
  v
Spring Boot API
  |
  v
Temporary upload URL
  |
  v
Client
  |
  v
Object Storage
```

The backend authorizes the upload while the client transfers the file directly to storage.

---

## 12. Pre-Signed URL

A pre-signed URL gives temporary permission to access an object.

Flow:

```text
Client
  |
  | "I want to upload photo.jpg"
  v
Backend
  |
  | Authorize
  v
Temporary URL
  |
  v
Client
  |
  v
Object Storage
```

Benefits:

```text
Less API bandwidth
Better scalability
Temporary access
Large-file support
```

---

## 13. Upload Security

For uploads:

```text
Validate size
Validate content
Validate type
Generate safe object names
Restrict permissions
Scan where appropriate
```

Do not trust the filename, extension, or Content-Type header alone.

---

## 14. CDN

A Content Delivery Network caches content closer to users.

Without CDN:

```text
User -> Origin
```

With CDN:

```text
User
 |
 v
CDN
 |
 +--> Cache hit
 |
 +--> Origin on miss
```

Useful for:

```text
Images
CSS
JavaScript
Videos
Downloads
Static assets
```

---

## 15. CDN Cache Hit

```text
User
 |
 v
CDN
 |
 v
Cached content
```

Origin is not contacted.

This reduces:

```text
Origin traffic
Latency
Bandwidth
```

---

## 16. CDN Cache Miss

```text
User
 |
 v
CDN
 |
 v
Origin
 |
 v
Content
 |
 v
CDN cache
 |
 v
User
```

Future requests can use the cached content.

---

## 17. Cache-Control

HTTP caching can be controlled with headers such as:

```http
Cache-Control: public, max-age=86400
```

Versioned static assets are especially useful:

```text
app.abc123.js
```

A changed file gets a new name, allowing long cache lifetimes.

---

## 18. Cache Invalidation

A common problem:

```text
Old object cached
      |
      v
Origin has new object
```

Strategies:

```text
Short TTL
Explicit invalidation
Versioned object names
Cache busting
```

Versioned filenames are often the simplest long-term approach for static assets.

---

## 19. Multipart Upload

Large files can be split into parts.

```text
10 GB file
    |
    +--> Part 1
    +--> Part 2
    +--> Part 3
    +--> ...
```

Benefits:

```text
Parallel uploads
Resume failed uploads
Avoid restarting the entire transfer
Better large-file performance
```

---

## 20. Resumable Upload

If a 10 GB upload fails at 9 GB:

Bad:

```text
Start again from 0
```

Better:

```text
Resume remaining parts
```

Important for:

```text
Videos
Large backups
Large datasets
```

---

## 21. Upload Completion

A safe workflow:

```text
Create upload
    |
    v
Upload parts
    |
    v
Verify parts
    |
    v
Complete upload
    |
    v
Mark file READY
```

Do not treat an incomplete upload as a usable file.

---

## 22. File Metadata in Database

The database can store:

```text
fileId
objectKey
ownerId
contentType
size
status
createdAt
```

Example:

```text
fileId = 101
objectKey = uploads/8f4c/report.pdf
status = READY
```

---

## 23. File Lifecycle

Useful states:

```text
UPLOADING
READY
PROCESSING
FAILED
DELETED
```

Example:

```text
Upload
  |
  v
UPLOADING
  |
  v
READY
```

---

## 24. Asynchronous File Processing

For image processing, scanning, or transcoding:

```text
Upload
  |
  v
Object Storage
  |
  v
Event / Queue
  |
  v
Worker
  |
  v
Process File
```

The API doesn't need to keep the request open during long processing.

---

## 25. Image Processing

```text
Original image
      |
      v
Object Storage
      |
      v
Queue
      |
      v
Image Worker
   /        v        v
Thumbnail  Optimized image
```

Generated variants can be stored separately.

---

## 26. Video Processing

A video platform might use:

```text
Upload
  |
  v
Object Storage
  |
  v
Queue
  |
  v
Transcoding Workers
  |
  +--> 360p
  +--> 720p
  +--> 1080p
```

A CDN can then serve the processed content.

---

## 27. Durability vs Availability

### Durability

Probability that stored data is not lost.

### Availability

Probability that data can be accessed when requested.

A storage system can be highly durable but temporarily unavailable.

---

## 28. Replication

Storage can replicate data:

```text
Object
 |
 +--> Replica A
 +--> Replica B
 +--> Replica C
```

Replication can improve resilience but introduces:

```text
Storage cost
Consistency considerations
Network traffic
```

---

## 29. Backup vs Replication

Replication is not a replacement for backups.

Replication:

```text
Primary changes
   |
   v
Replica changes
```

If data is accidentally deleted, that deletion may also replicate.

Backups provide historical recovery points.

---

## 30. Versioning

Object versioning can preserve previous versions:

```text
report.pdf
   |
   +--> v1
   +--> v2
   +--> v3
```

Useful for:

```text
Accidental deletion
Accidental overwrite
Recovery
Audit requirements
```

Trade-off:

```text
More storage
More lifecycle management
```

---

## 31. Soft Delete

Instead of immediately deleting:

```text
deleted = true
```

or:

```text
status = DELETED
```

Benefits:

```text
Recovery
Auditability
Accidental-delete protection
```

A background process can permanently delete old data later.

---

## 32. Lifecycle Policies

Storage can automatically transition or delete objects.

Example:

```text
0-30 days  -> Standard
30-90 days -> Infrequent Access
90+ days   -> Archive
```

This reduces storage cost for rarely accessed data.

---

## 33. Hot vs Cold Data

### Hot data

Frequently accessed:

```text
Product images
Recent reports
Active documents
```

### Cold data

Rarely accessed:

```text
Old backups
Archives
Historical logs
```

Cold storage can reduce cost at the expense of retrieval latency and/or fees.

---

## 34. Storage Cost

Consider:

```text
Storage capacity
Requests
Data transfer
Replication
Backups
Retrieval
CDN
Processing
```

A cheap storage tier can become expensive if data is frequently retrieved.

---

## 35. Egress

Data transfer out of a storage system or cloud environment may incur cost.

Example:

```text
Object Storage
      |
      v
Internet
      |
      v
Millions of users
```

A CDN can reduce origin traffic and improve delivery economics depending on the architecture.

---

## 36. Access Control

Object access should be controlled using:

```text
Identity
Role
Resource ownership
Policy
Expiration
```

Avoid making sensitive objects public simply to simplify downloads.

---

## 37. Signed Download URL

For private files:

```text
Client
  |
  v
Backend
  |
  | Authorize
  v
Temporary download URL
  |
  v
Object Storage
```

The URL can expire after a short period.

---

## 38. Checksums

Checksums can verify that a file transferred correctly.

```text
Original
   |
   v
Checksum A

Uploaded
   |
   v
Checksum B
```

If they match, the content is likely unchanged under the checksum's guarantees.

---

## 39. Deduplication

If users upload identical files, the system can potentially store one physical object and reference it multiple times.

Possible approach:

```text
Content hash
   |
   v
Existing object?
```

Trade-offs:

```text
Hash computation
Reference management
Security/privacy concerns
```

---

## 40. Safe File Naming

Avoid directly using user-provided names as storage paths.

Bad:

```text
../../important-file
```

Better:

```text
Generated object ID
+
Validated metadata
```

---

## 41. File System vs Object Storage

| File System | Object Storage |
|---|---|
| Directory/file semantics | Bucket/key semantics |
| Good for shared files | Good for large unstructured data |
| Traditional applications | Cloud-native/static content |
| Filesystem APIs | HTTP/API-oriented |

---

## 42. Block vs File vs Object

| Type | Best For |
|---|---|
| Block | VM disks, databases |
| File | Shared directories, traditional applications |
| Object | Images, videos, backups, documents |

Interview shortcut:

> "Block storage is low-level storage, file storage provides filesystem semantics, and object storage is optimized for scalable unstructured objects."

---

## 43. E-commerce Storage Architecture

```text
                 Client
                    |
                    v
                  CDN
                    |
                    v
              Object Storage
                    ^
                    |
               Product Images

Client
   |
   v
Spring Boot API
   |
   +--> MySQL
   |     |
   |     +--> Product metadata
   |     +--> Order data
   |
   +--> Redis
```

Database:

```text
Business metadata
```

Object storage:

```text
Large files
```

Redis:

```text
Caching
```

CDN:

```text
Global content delivery
```

---

## 44. Video Platform Architecture

```text
                 Upload
                    |
                    v
             Object Storage
                    |
                    v
                  Queue
                    |
                    v
             Transcoding Workers
               /     |                    v      v       v
            360p    720p    1080p
               \      |      /
                v     v     v
                  CDN
                    |
                    v
                  Users
```

---

## 45. Document Storage Architecture

```text
Client
  |
  v
API
  |
  +--> Authorization
  |
  +--> File Metadata DB
  |
  +--> Pre-signed URL
          |
          v
      Object Storage
```

Private documents should not simply be stored in publicly readable storage.

---

## 46. Range Requests

HTTP range requests allow clients to request only part of a file.

Useful for:

```text
Video seeking
Large downloads
Resume support
```

Conceptually:

```text
GET bytes 10MB-20MB
```

---

## 47. Storage Failure

If storage becomes temporarily unavailable:

```text
API
 |
 v
Storage X
```

Possible strategies:

```text
Timeouts
Bounded retries
Queue uploads
Return temporary-unavailable state
Replicated storage where appropriate
```

Do not retry forever.

---

## 48. Orphaned Objects

Example:

```text
Object uploaded
     |
     X
Database transaction fails
```

Now:

```text
Object exists
DB record doesn't
```

Solutions:

```text
File status workflow
Periodic reconciliation
Lifecycle cleanup
Delayed deletion
```

---

## 49. Orphaned Metadata

The opposite can happen:

```text
DB record exists
Object missing
```

Use:

```text
Validation
Reconciliation jobs
Storage monitoring
Alerts
```

Critical systems should detect mismatches automatically.

---

## 50. Temporary Uploads

Use a temporary state/path:

```text
UPLOADING
```

If an upload is abandoned:

```text
Cleanup after TTL
```

This prevents storage leaks.

---

## 51. Backup Strategy

Define:

```text
Backup frequency
Retention
Encryption
Geographic separation
Restore process
Restore testing
```

Remember:

> A backup that has never been restored is not fully trusted.

---

## 52. RTO and RPO

### RTO

How quickly the system must recover.

```text
RTO = 30 minutes
```

### RPO

How much data loss measured in time is acceptable.

```text
RPO = 5 minutes
```

Storage architecture should support the required RTO/RPO.

---

## 53. Interview — Why Use Object Storage?

> "I'd use object storage for large unstructured data such as images, videos and documents because it scales independently from the application database and is designed for large amounts of data and high durability. The relational database can store metadata and object references."

---

## 54. Interview — Why Use Pre-Signed URLs?

> "Pre-signed URLs allow the client to upload or download directly from object storage for a limited time. This reduces API bandwidth and makes large-file transfers more scalable while still allowing the backend to authorize access."

---

## 55. Interview — How Would You Design File Uploads?

> "I'd store file metadata in the database and the actual file in object storage. For large files, I'd use pre-signed URLs and multipart/resumable uploads. After upload, I'd validate or scan the file and update its status. Asynchronous processing can be triggered through a queue."

---

## 56. Interview — Why Use a CDN?

> "A CDN caches content close to users, reducing latency and origin traffic. I'd use it for static assets, images, downloads and video delivery, while keeping authorization and business logic in the backend."

---

## 57. Interview — Database or Object Storage for Images?

> "For most large images and media, I'd use object storage and keep the object key and metadata in the database. Storing large binaries in the database can increase database size, backup cost and I/O."

---

## 58. Interview — How Do You Handle Large File Uploads?

> "I'd avoid proxying the entire file through the Spring Boot application. I'd generate a temporary upload URL and let the client upload directly to object storage, using multipart or resumable uploads for very large files."

---

## 59. Practical Scenario — 10 GB Upload

Recommended:

```text
Client
  |
  v
Backend -> temporary upload permission
  |
  v
Object Storage
  ^
  |
Multipart upload
```

Benefits:

```text
Parallelism
Resume support
Less API bandwidth
Better reliability
```

---

## 60. Practical Scenario — User Deletes a File

Safer flow:

```text
DELETE request
     |
     v
Authorization
     |
     v
Mark metadata DELETED
     |
     v
Delete object asynchronously
```

For critical data, consider a soft-delete/recovery period before permanent deletion.

---

## 61. Practical Scenario — DB and Object Storage Disagree

Example:

```text
DB -> file READY
Object -> missing
```

Use:

```text
Reconciliation job
Storage validation
Audit logs
Alerts
```

---

## 62. Practical Scenario — CDN Serves Old Image

Possible fixes:

```text
Versioned filenames
Cache invalidation
Shorter TTL
Cache-Control headers
```

Versioned assets are often the cleanest long-term solution.

---

## 63. Practical Scenario — Millions of Users Download Videos

Architecture:

```text
Users
  |
  v
CDN
  |
  v
Object Storage
```

Don't send every video byte through Spring Boot.

Use:

```text
CDN caching
Range requests
Adaptive media formats where appropriate
Object storage
```

---

## 64. Final Checklist

```text
□ Block storage
□ File storage
□ Object storage
□ Object keys
□ Object metadata
□ Database vs object storage
□ Direct uploads
□ Pre-signed URLs
□ Upload security
□ CDN
□ Cache hit/miss
□ Cache-Control
□ Cache invalidation
□ Multipart uploads
□ Resumable uploads
□ File lifecycle states
□ Async file processing
□ Image/video processing
□ Durability vs availability
□ Replication
□ Backup vs replication
□ Object versioning
□ Soft delete
□ Lifecycle policies
□ Hot vs cold data
□ Storage cost
□ Egress
□ Access control
□ Signed download URLs
□ Checksums
□ Deduplication
□ Safe filenames
□ Range requests
□ Storage consistency
□ Storage failure handling
□ Orphaned objects
□ Orphaned metadata
□ Reconciliation
□ Temporary uploads
□ Backup/restore
□ RTO/RPO
```

---

## 65. One-Minute Interview Answer

### "How would you design file storage for an e-commerce application?"

> "I'd keep product and file metadata in MySQL and store the actual images or documents in object storage. For uploads, the backend would authenticate and authorize the request, generate a short-lived pre-signed URL, and let the client upload directly to object storage. For large files I'd use multipart uploads. A CDN would serve frequently accessed images close to users. I'd also validate uploads, control access, maintain file lifecycle states, clean up orphaned objects and define backup and retention policies."

---

## 66. Key Takeaway

> **Use the right storage for the data. Keep business metadata in the database, large unstructured files in object storage, and use CDNs and direct uploads to scale file delivery without turning the application server into a file-transfer bottleneck.**

**File 18 complete.**
