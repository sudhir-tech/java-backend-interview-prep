# System Design — File 20: Real-Time Systems

Real-time systems deliver updates with very low delay instead of requiring the client to repeatedly request the latest state.

Examples:

```text
Chat
Notifications
Live sports scores
Trading dashboards
Delivery tracking
Online presence
Collaborative editing
```

The right communication mechanism depends on latency, direction of communication, reliability and scale.

---

## 1. Real-Time Communication Options

Common approaches:

```text
Polling
Long Polling
Server-Sent Events
WebSockets
Pub/Sub
```

They solve different problems.

---

## 2. Polling

The client repeatedly asks the server:

```text
Client -> GET /notifications
Client -> GET /notifications
Client -> GET /notifications
```

### Advantages

```text
Simple
Easy to implement
Works with normal HTTP
```

### Disadvantages

```text
Unnecessary requests
Higher latency
More server load
Poor real-time behavior
```

Polling is acceptable when updates are infrequent or simplicity matters more than latency.

---

## 3. Long Polling

The client sends a request:

```text
Client
  |
  v
Server
  |
  | wait for update
  |
  v
Response
```

After receiving the response, the client sends another request.

Compared with normal polling:

```text
Fewer unnecessary responses
Lower latency
```

But connections are still repeatedly created and managed.

---

## 4. Server-Sent Events

SSE allows the server to continuously send events over an HTTP connection.

```text
Client
  |
  | HTTP connection
  v
Server
  |
  +--> Event 1
  +--> Event 2
  +--> Event 3
```

Good for:

```text
Notifications
Live dashboards
Progress updates
Server-to-browser events
```

---

## 5. SSE Is Server-to-Client

SSE primarily provides:

```text
Server -> Client
```

The client still sends normal HTTP requests:

```text
Client -> Server
```

This makes SSE simpler than WebSockets for many one-way streaming use cases.

---

## 6. WebSockets

WebSockets provide a persistent two-way connection.

```text
Client <=================> Server
       bidirectional
```

Both sides can send messages at any time.

Useful for:

```text
Chat
Multiplayer games
Collaborative applications
Real-time tracking
Trading interfaces
Presence
```

---

## 7. WebSocket Handshake

WebSockets begin with an HTTP request and upgrade to a WebSocket connection.

Conceptually:

```text
HTTP Request
     |
     v
Upgrade
     |
     v
WebSocket Connection
```

After the upgrade, messages can flow in both directions.

---

## 8. Polling vs Long Polling vs SSE vs WebSocket

| Method | Direction | Persistent | Good For |
|---|---|---|---|
| Polling | Client-driven | No | Simple updates |
| Long Polling | Mostly server response | Temporary | Near-real-time |
| SSE | Server -> Client | Yes | Notifications/streams |
| WebSocket | Bidirectional | Yes | Chat/interactive real-time |

---

## 9. When to Use WebSockets

Use WebSockets when:

```text
Low latency matters
Server needs to push updates
Client also sends frequent messages
Bidirectional communication is required
```

For example:

```text
Chat message
Typing indicator
Online presence
Read receipt
```

---

## 10. When SSE Is Better

Use SSE when:

```text
Server -> client updates dominate
HTTP compatibility is valuable
Communication is mostly one-way
```

Examples:

```text
Notification feed
Live dashboard
Job progress
Server status stream
```

Don't choose WebSockets automatically just because the application is called "real-time."

---

## 11. Connection Lifecycle

A real-time connection can be:

```text
CONNECTING
     |
     v
CONNECTED
     |
     v
DISCONNECTING
     |
     v
DISCONNECTED
```

The client should handle:

```text
Connection failure
Reconnect
Backoff
Authentication expiry
Network changes
```

---

## 12. Heartbeats

Long-lived connections can become stale.

Use:

```text
Ping
Pong
```

or application-level heartbeat messages.

Conceptually:

```text
Client -> Ping
Server -> Pong
```

If the connection becomes unresponsive:

```text
Close
Reconnect
```

---

## 13. Reconnection

A client should not reconnect thousands of times immediately.

Bad:

```text
Failure
 |
 +--> reconnect
 +--> reconnect
 +--> reconnect
```

Better:

```text
Failure
 |
 v
Backoff
 |
 v
Retry
 |
 v
Longer backoff
```

Use exponential backoff with jitter.

---

## 14. Reconnection and Message Loss

Suppose:

```text
Client disconnected
```

Events may occur while it is offline.

The system needs a strategy:

```text
Replay missed events
Fetch current state
Use durable message storage
```

For many applications, reconnecting and fetching the latest state is simpler than replaying every event.

---

## 15. Connection State

Real-time systems may track:

```text
User
Connection ID
Device
Session
Last activity
```

Example:

```text
User 42
  |
  +--> Laptop connection
  +--> Mobile connection
```

A user may have multiple simultaneous connections.

---

## 16. Presence

Presence answers:

```text
Is the user online?
```

Possible states:

```text
ONLINE
AWAY
OFFLINE
```

But presence is usually approximate.

A network failure can prevent the server from immediately knowing that a client disappeared.

---

## 17. Presence with Heartbeats

```text
Client
  |
  | heartbeat
  v
Presence Service
  |
  v
lastSeen = now
```

If no heartbeat arrives for a timeout:

```text
Mark offline
```

Use a timeout rather than assuming immediate disconnect detection.

---

## 18. Redis for Presence

A common approach:

```text
Redis
 |
 +--> user:42 -> online
 +--> user:43 -> offline
```

Or store last-seen timestamps.

Redis is useful because:

```text
Low latency
TTL support
Shared state across instances
```

---

## 19. Scaling WebSockets

One server:

```text
Clients
   |
   v
WebSocket Server
```

At scale:

```text
              Load Balancer
             /      |                  v       v       v
          WS1     WS2     WS3
```

The challenge is that a user's connection exists on one specific server.

---

## 20. Sticky Sessions

A load balancer can route the same client back to the same server.

```text
User 42
   |
   v
Load Balancer
   |
   v
WS2
```

Benefits:

```text
Simple connection management
```

Problems:

```text
Uneven load
Server failure disconnects clients
State remains tied to instances
```

Sticky sessions are not always the best long-term solution.

---

## 21. Shared Pub/Sub

Instead of relying on one WebSocket server:

```text
              Redis Pub/Sub
              /     |                  v      v      v
           WS1    WS2    WS3
```

If a message arrives at WS1:

```text
WS1
 |
 v
Redis Pub/Sub
 |
 +--> WS1
 +--> WS2
 +--> WS3
```

Each relevant server can deliver the message to its connected clients.

---

## 22. Pub/Sub vs Durable Messaging

Redis Pub/Sub is generally ephemeral.

If a subscriber is disconnected:

```text
Message
  |
  v
No subscriber
  |
  v
Message may be missed
```

For durable replay requirements, use a durable messaging system such as:

```text
Kafka
```

or another suitable durable broker.

---

## 23. WebSocket Architecture

A scalable architecture:

```text
                    Load Balancer
                   /      |                        v       v       v
                WS1     WS2     WS3
                  \       |       /
                   \      |      /
                    v     v     v
                   Redis / Broker
                         |
                         v
                    Application
                         |
                         v
                       MySQL
```

WebSocket servers manage connections.

The application layer owns business state.

---

## 24. Chat Message Flow

Example:

```text
User A
  |
  v
WebSocket Server
  |
  v
Chat Service
  |
  +--> Database
  |
  +--> Message Broker
  |
  v
WebSocket Server
  |
  v
User B
```

Persist important messages before considering them delivered, depending on product requirements.

---

## 25. Message States

A chat message can have:

```text
SENDING
SENT
DELIVERED
READ
FAILED
```

Example:

```text
SENT
  |
  v
DELIVERED
  |
  v
READ
```

These states should be modeled explicitly when the product needs them.

---

## 26. Delivery Semantics

Real-time messages can be:

```text
At-most-once
At-least-once
Effectively-once
```

For important messages:

```text
Message ID
+
Deduplication
+
Durable storage
```

can prevent duplicate business effects.

---

## 27. Message Ordering

Suppose:

```text
Message A
Message B
```

Network conditions can cause:

```text
B arrives
A arrives
```

If ordering matters, use:

```text
Sequence number
Conversation-specific ordering
Partitioning
Server-side ordering
```

Don't assume network delivery automatically preserves business order.

---

## 28. Sequence Numbers

Example:

```text
Conversation 42

seq=100 -> Hello
seq=101 -> How are you?
seq=102 -> Good morning
```

If the client receives:

```text
100
102
```

it knows:

```text
101 is missing
```

It can request recovery.

---

## 29. Offline Users

If User B is offline:

```text
User A -> Message -> Server
                  |
                  v
              Database
```

When User B reconnects:

```text
Reconnect
   |
   v
Fetch missed messages
```

This is usually more reliable than depending entirely on transient WebSocket delivery.

---

## 30. Notification System

A scalable notification architecture:

```text
Business Event
      |
      v
Message Broker
      |
      v
Notification Service
      |
      +--> Push
      +--> Email
      +--> SMS
      +--> In-App
```

The notification service can process channels independently.

---

## 31. In-App Notifications

```text
Backend
   |
   v
Notification DB
   |
   v
WebSocket / SSE
   |
   v
Browser
```

Persist the notification before pushing it if the user needs it to survive disconnects.

---

## 32. Push Notifications

Mobile applications may use platform push services.

Conceptually:

```text
Backend
   |
   v
Push Provider
   |
   v
Mobile Device
```

The backend should not assume that a push message guarantees that the user saw it.

---

## 33. Live Tracking

Example:

```text
Delivery Driver
      |
      v
Location Service
      |
      v
Event Stream
      |
      v
Customer WebSocket
      |
      v
Map
```

Location updates should be throttled appropriately.

Do not send every GPS point if the client doesn't need that frequency.

---

## 34. Rate Limiting Real-Time Connections

Attackers can create huge numbers of connections.

Protect with:

```text
Connection limits
Per-user limits
Per-IP limits
Authentication
Rate limits
Resource quotas
```

---

## 35. Authentication

Authenticate WebSocket/SSE connections.

For example:

```text
Client
  |
  v
Connection request
  |
  v
Authentication
  |
  v
Established connection
```

Do not treat a WebSocket connection as trusted simply because it was established.

---

## 36. Authorization

A connected user should only receive events they are allowed to see.

Example:

```text
User A
   |
   v
Order 101
```

The server must not send:

```text
Order 202 belonging to User B
```

to the same connection.

---

## 37. Tenant Isolation

For multi-tenant systems:

```text
Tenant A
   |
   +--> Users
   +--> Events

Tenant B
   |
   +--> Users
   +--> Events
```

Every event should be scoped to the correct tenant.

---

## 38. Backpressure

What happens if the client is slower than the server?

```text
Server
  |
  | 10,000 events/sec
  v
Client
  |
  | 100 events/sec
```

The buffer grows.

Eventually:

```text
Memory exhaustion
```

Solutions:

```text
Bounded buffers
Drop non-critical updates
Batch events
Throttle producers
Disconnect slow clients
Send latest state instead of every event
```

---

## 39. Latest-State Optimization

For rapidly changing values such as:

```text
Driver location
Stock price
CPU usage
```

you may not need every intermediate update.

Instead:

```text
Old state -> discard
Latest state -> deliver
```

This reduces traffic significantly.

---

## 40. Connection Limits

A real-time server has limits based on:

```text
File descriptors
Memory
CPU
Network
TLS overhead
Connection state
```

Monitor:

```text
Active connections
Connection creation rate
Disconnect rate
Memory per connection
Messages/sec
```

---

## 41. Load Balancing WebSockets

The load balancer must support long-lived connections.

Architecture:

```text
Client
  |
  v
Load Balancer
  |
  +--> WS1
  +--> WS2
  +--> WS3
```

Health checks should remove unhealthy instances.

---

## 42. Reconnection Storm

A server crashes:

```text
WS1 X
 |
 v
100,000 clients disconnect
 |
 v
100,000 reconnect immediately
 |
 v
WS2 / WS3 overloaded
```

This is a reconnection storm.

Prevent with:

```text
Exponential backoff
Jitter
Connection admission control
Rate limiting
Gradual recovery
```

---

## 43. Graceful WebSocket Shutdown

Before shutting down:

```text
Stop accepting new connections
        |
        v
Notify existing clients
        |
        v
Allow graceful close
        |
        v
Drain connections
```

Clients reconnect to healthy instances.

---

## 44. Real-Time Data vs Durable Data

Separate:

```text
Connection delivery
```

from:

```text
Business persistence
```

For example:

```text
WebSocket -> fast delivery
Database  -> durable history
Kafka     -> event distribution
```

This separation makes the architecture more resilient.

---

## 45. WebSocket + Kafka

A common large-scale pattern:

```text
Business Service
      |
      v
Kafka
      |
      v
Realtime Gateway
      |
      v
WebSocket Clients
```

Kafka provides durable event distribution.

The real-time gateway handles:

```text
Connection management
Subscription
Authorization
Delivery
```

---

## 46. Subscription Model

Clients can subscribe to topics:

```text
user:42
order:101
chat:room123
stock:AAPL
```

The server checks authorization before allowing subscription.

---

## 47. Fan-Out

One event may need to reach many clients.

```text
New Post
   |
   v
Fan-Out
 / | v  v  v
U1 U2 U3
```

For millions of recipients, synchronous delivery from one request is not practical.

Use:

```text
Queues
Workers
Batched delivery
Push infrastructure
```

---

## 48. Fan-Out on Write vs Read

### Fan-Out on Write

When content is created:

```text
Create post
   |
   v
Prepare feeds for followers
```

Fast reads, more write work.

### Fan-Out on Read

When user opens feed:

```text
Read followed users
   |
   v
Build feed
```

Less write work, more read work.

Choose based on workload.

---

## 49. Real-Time Feed

For a social feed:

```text
Post Created
     |
     v
Event
     |
     v
Feed Service
     |
     v
Redis / Feed Store
     |
     v
Client
```

For huge follower counts, special handling may be needed for celebrity accounts.

---

## 50. Presence at Scale

A user can have:

```text
Laptop
Mobile
Tablet
```

Presence should be aggregated:

```text
User online if
at least one valid connection exists
```

Track connections rather than simply setting:

```text
user=online
```

on every connection.

---

## 51. Connection Registry

Example:

```text
User 42
 |
 +--> conn-A -> WS1
 +--> conn-B -> WS3
```

When one connection closes:

```text
conn-A removed
```

User remains online if:

```text
conn-B still active
```

---

## 52. Redis TTL for Presence

A simple approach:

```text
user:42:presence
TTL = 30 seconds
```

Heartbeat refreshes the TTL.

If no heartbeat:

```text
Key expires
```

The user can be considered offline.

---

## 53. Real-Time Security

Protect against:

```text
Unauthorized subscriptions
Connection exhaustion
Message injection
Cross-tenant data leakage
Token theft
Replay
```

Use:

```text
Authentication
Authorization
TLS
Rate limiting
Input validation
Connection quotas
```

---

## 54. Observability

Monitor:

```text
Active connections
Connection failures
Reconnect rate
Messages/sec
Message latency
Dropped messages
Slow consumers
Queue depth
CPU
Memory
Network
```

For chat:

```text
Message send latency
Delivery latency
Read-receipt latency
```

---

## 55. Real-Time System Trade-Offs

More real-time behavior often means:

```text
More connections
More network traffic
More infrastructure
More state management
More failure scenarios
```

Don't make everything real-time unnecessarily.

---

## 56. Interview — WebSocket vs SSE?

> "I'd use WebSockets when communication needs to be bidirectional and low latency, such as chat. I'd use SSE when updates are primarily server-to-client, such as notifications or live dashboards, because it is simpler and works over normal HTTP."

---

## 57. Interview — How Do You Scale WebSockets?

> "I'd put WebSocket servers behind a load balancer and keep connection state lightweight. For cross-instance message delivery, I'd use a shared pub/sub or durable broker such as Redis or Kafka depending on whether replay is required. I'd also handle heartbeats, reconnection with backoff, connection limits and authorization."

---

## 58. Interview — How Do You Handle Offline Users?

> "I'd persist important messages or notifications in durable storage. When the client reconnects, it can fetch the messages it missed. The WebSocket is primarily the low-latency delivery channel, not the only source of truth."

---

## 59. Interview — How Do You Handle Message Ordering?

> "I'd assign sequence numbers within the required ordering scope, such as a conversation. The client can detect missing sequence numbers and request recovery. I wouldn't assume network delivery automatically preserves business ordering."

---

## 60. Interview — How Do You Prevent Reconnection Storms?

> "I'd use exponential backoff with jitter on the client, plus connection admission controls and rate limiting on the server. That spreads reconnect attempts instead of allowing every disconnected client to reconnect simultaneously."

---

## 61. Practical Scenario — 1 Million WebSocket Connections

Consider:

```text
Load Balancer
      |
      +--> WS1
      +--> WS2
      +--> WS3
      +--> ...
```

Scale horizontally.

Monitor:

```text
Connections per node
Memory
CPU
Network
Messages/sec
Disconnect rate
```

Use shared event distribution so clients connected to different nodes can receive relevant messages.

---

## 62. Practical Scenario — Chat Server Crashes

Clients:

```text
Disconnected
```

Use:

```text
Client reconnect with backoff
Server authentication
Fetch missed messages
Resume normal WebSocket delivery
```

Persist messages so the connection itself is not the only copy.

---

## 63. Practical Scenario — Slow Client

If:

```text
Producer = 10,000 msg/sec
Client = 100 msg/sec
```

Don't allow an unlimited buffer.

Use:

```text
Bounded queue
Drop intermediate state where safe
Batch
Throttle
Disconnect very slow clients
```

---

## 64. Practical Scenario — User Has Multiple Devices

Track:

```text
User -> Connection IDs
```

Then:

```text
Message
   |
   +--> Mobile
   +--> Laptop
```

depending on product requirements.

---

## 65. Practical Scenario — Live Driver Location

Use:

```text
Driver
  |
  v
Location Service
  |
  v
Event Stream
  |
  v
Realtime Gateway
  |
  v
Customer
```

Optimize by:

```text
Throttling updates
Sending latest state
Using geographic filtering
```

Do not send every raw GPS update to every customer.

---

## 66. Final Checklist

```text
□ Polling
□ Long polling
□ SSE
□ WebSockets
□ Choosing the right protocol
□ WebSocket handshake
□ Connection lifecycle
□ Heartbeats
□ Reconnection
□ Reconnection backoff
□ Message loss
□ Connection state
□ Presence
□ Redis presence
□ Scaling WebSockets
□ Sticky sessions
□ Pub/Sub
□ Durable messaging
□ Chat architecture
□ Message states
□ Delivery semantics
□ Message ordering
□ Sequence numbers
□ Offline users
□ Notifications
□ Push notifications
□ Live tracking
□ Connection rate limiting
□ Authentication
□ Authorization
□ Tenant isolation
□ Backpressure
□ Connection limits
□ Load balancing
□ Reconnection storms
□ Graceful shutdown
□ Durable vs real-time state
□ Kafka + WebSockets
□ Subscriptions
□ Fan-out
□ Fan-out on write/read
□ Feed architecture
□ Presence at scale
□ Redis TTL
□ Security
□ Observability
□ Real-time trade-offs
```

---

## 67. One-Minute Interview Answer

### "How would you design a real-time chat system?"

> "I'd use WebSockets for bidirectional low-latency communication. WebSocket servers would sit behind a load balancer, and I'd use a shared event layer such as Redis Pub/Sub or Kafka so users connected to different servers can communicate. Important messages would be persisted in a database, because WebSocket delivery alone isn't durable. I'd use message IDs and sequence numbers for deduplication and ordering, heartbeats and exponential-backoff reconnects for connection reliability, and authentication and per-conversation authorization for security. I'd also monitor active connections, message latency, reconnect rate and slow consumers."

---

## 68. Key Takeaway

> **Real-time systems separate fast delivery from durable state. WebSockets are ideal for bidirectional low-latency communication, SSE is often simpler for server-to-client streams, and durable storage or messaging should handle recovery, replay and business correctness.**

**File 20 complete.**
