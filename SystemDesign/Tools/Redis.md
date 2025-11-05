

# **Redis**

**Redis (REmote DIctionary Server)** is an **in-memory data structure store**, used primarily as:

* A **database**
* A **cache**
* A **message broker**

It supports multiple advanced data types beyond simple key-value pairs, and provides **persistence, replication, clustering, transactions, and Lua scripting**.

* **Language:** C
* **Latency:** < 1ms typical
* **Model:** Single-threaded event-driven
* **License:** BSD
* **Network Protocol:** RESP (REdis Serialization Protocol)

---

## 2. Core Architecture

Redis follows a **single-threaded event loop architecture** — all commands are processed sequentially, but I/O (networking, persistence) uses efficient non-blocking techniques (epoll/kqueue).

### Major Components:

| Component                    | Description                                                                             |
| ---------------------------- | --------------------------------------------------------------------------------------- |
| **Main thread (Event loop)** | Handles client connections, requests, and responses sequentially using I/O multiplexing |
| **Persistence layer**        | RDB and AOF modules for disk persistence                                                |
| **Replication module**       | Manages master-replica data sync                                                        |
| **Pub/Sub system**           | Enables real-time message broadcasting                                                  |
| **Cluster manager**          | Handles data sharding and cluster state                                                 |
| **Memory allocator**         | Uses `jemalloc` for fast memory allocation                                              |

---

## 3. Data Structures & Internal Representation

Redis provides **high-level data types**, each optimized internally.

| Type                           | Description                         | Internal Encoding                    |
| ------------------------------ | ----------------------------------- | ------------------------------------ |
| **String**                     | Binary-safe, up to 512 MB           | `raw`, `embstr`, `int`               |
| **List**                       | Linked list of strings              | `quicklist` (compressed linked list) |
| **Set**                        | Unordered unique strings            | `intset` (if small), `hashtable`     |
| **Hash**                       | Key-value pairs within key          | `ziplist` (small) or `hashtable`     |
| **Sorted Set (ZSet)**          | Unique elements with scores         | `ziplist` or `skiplist`              |
| **Stream**                     | Log-like append-only structure      | Radix-tree + listpack                |
| **Bitmap / HyperLogLog / Geo** | Specialized probabilistic/geo types | Custom compact encodings             |

---

## 4. Execution Flow

### Request Lifecycle:

1. Client sends command via TCP using RESP.
2. Event loop reads request (non-blocking I/O via `epoll`).
3. Command is parsed and executed sequentially.
4. Response encoded and sent back via event loop.
5. If persistence is enabled, changes are logged asynchronously.

### Why single-threaded?

* Avoids context-switching overhead.
* Relies on fast in-memory ops.
* Uses multiplexed non-blocking I/O (handles tens of thousands of clients).

### Multi-threading (since Redis 6+)

* For **I/O operations only** (network reads/writes, not command execution).
* Controlled by `io-threads-do-reads yes` and `io-threads <num>`.

---

## 5. Persistence (Durability Mechanisms)

Redis keeps data in memory but can persist it using **RDB** or **AOF** (or both).

### **RDB (Redis Database File)**

* Snapshot-based persistence.
* Dumps all in-memory data to disk periodically (`SAVE` or `BGSAVE`).
* Compact binary file.
* Fast restart, but data loss possible between snapshots.

**Process:**

* `fork()` → child process writes DB snapshot to temp file → atomically renames.

### **AOF (Append Only File)**

* Logs every write operation.
* Can be configured to `fsync` every write / second / never.
* Slower but more durable.
* Can be rewritten periodically to compact file.

**Rewrite process:**
`fork()` → child writes a compact AOF while parent appends new commands to buffer → merged at the end.

### **Hybrid persistence (RDB + AOF, Redis 7+)**

* Combines fast recovery (RDB) with durability (AOF tail).

---

## 6. Replication (Master–Replica Model)

Redis supports **asynchronous replication** with optional partial resynchronization.

### **Workflow:**

1. Replica connects to master.
2. Full sync if first time (RDB sent).
3. Incremental sync using replication backlog buffer.
4. Replica acknowledges received offsets.

### **Key Concepts:**

* **Replication backlog buffer:** Fixed-size circular buffer for master’s latest write commands.
* **PSYNC command:** Allows partial resync.
* **Replica can be read-only** (default).

### **Topology:**

* One master → multiple replicas.
* Replicas can replicate from replicas (Redis 5+).

---

## 7. High Availability (Sentinel)

Redis Sentinel provides **automatic failover and monitoring**.

### Sentinel components:

* **Monitoring:** Tracks master and replica health.
* **Notification:** Notifies admins or scripts.
* **Failover:** Promotes replica to master if needed.
* **Configuration provider:** Clients query Sentinel to discover new master.

Uses **Raft-like quorum** to decide failover (requires majority).

---

## 8. Clustering (Sharding)

Redis Cluster enables **horizontal scaling** via **sharding and replication**.

### **Architecture:**

* **16384 hash slots** distributed across masters.
* Each key’s slot = `CRC16(key) % 16384`.
* Each master holds subset of slots; replicas for redundancy.

### **Features:**

* Automatic data rebalancing.
* Multi-master replication.
* Redirects using `MOVED`/`ASK` responses.
* Consistent hashing avoided; slot-based approach ensures easy rebalancing.

### **Limitations:**

* Multi-key operations only allowed within same slot.
* Some commands restricted.

---

## 9. Eviction Policies (Memory Management)

If `maxmemory` is set, Redis applies an **eviction policy** when full.

### Policies:

| Policy            | Description                       |
| ----------------- | --------------------------------- |
| `noeviction`      | Return error on writes            |
| `allkeys-lru`     | Evict least recently used         |
| `volatile-lru`    | Evict LRU only from keys with TTL |
| `allkeys-random`  | Evict random keys                 |
| `volatile-random` | Evict random keys with TTL        |
| `volatile-ttl`    | Evict key with nearest expiry     |

Redis uses an **approximate LRU/LFU algorithm** for efficiency.

---

## 10. Pub/Sub & Streams

### **Pub/Sub**

* Real-time message broadcasting.
* Non-persistent.
* Commands: `PUBLISH`, `SUBSCRIBE`, `PSUBSCRIBE`, etc.

### **Streams (introduced in Redis 5)**

* Persistent, append-only log.
* Supports consumer groups, message IDs, acknowledgment.
* Suitable for event sourcing / queue systems.

Internal encoding: radix tree + listpack.

---

## 11. Transactions & Scripting

### **Transactions**

* Commands: `MULTI`, `EXEC`, `DISCARD`, `WATCH`.
* Commands queued and executed atomically.
* `WATCH` enables optimistic locking.

### **Lua scripting**

* Executes scripts atomically.
* Scripts cached via SHA1 hash.
* Sandbox environment (no system calls).

---

## 12. Modules & Extensions

Redis supports **loadable modules** (C/C++) to extend functionality.

Examples:

* `RedisJSON` – JSON data type
* `RediSearch` – full-text search
* `RedisGraph` – graph database
* `RedisTimeSeries` – time-series data

Modules can:

* Define new data types.
* Register custom commands.
* Hook into events and replication.

---

## 13. Networking & Protocol

### **RESP (Redis Serialization Protocol)**

* Simple text-based protocol.
* Examples:

  ```
  *2
  $3
  GET
  $3
  key
  ```
* RESP3 (Redis 6+) adds native types: maps, sets, nulls, streamed data.

### **Event-driven I/O**

* Based on `ae.c` (abstract event loop).
* Uses `epoll`, `kqueue`, or `select` depending on OS.

---

## 14. Performance Optimizations

* In-memory operations (RAM only).
* Pipelining: batch commands to reduce RTT.
* I/O threads for parallel network handling.
* Zero-copy replies (`sendfile()`).
* Data locality: key hashing → same slot.

Benchmark: > 1 million ops/sec on modern hardware.

---

## 15. Common Use Cases

| Use Case                  | How Redis Helps               |
| ------------------------- | ----------------------------- |
| **Caching layer**         | Fast retrieval, TTL, eviction |
| **Session storage**       | Expirable keys                |
| **Leaderboards**          | Sorted sets                   |
| **Message queues**        | Lists / Streams               |
| **Pub/Sub notifications** | Real-time communication       |
| **Rate limiting**         | Counters + Lua scripts        |
| **Geospatial data**       | Geo commands                  |

---

## 16. Security & Access Control

* **ACLs (Access Control Lists)**: since Redis 6.
* Commands: `ACL SETUSER`, `ACL LIST`.
* **TLS/SSL support**: since Redis 6.
* Protected mode by default (localhost only).

---

## 17. Observability

* **INFO** command: server metrics.
* **MONITOR**: real-time command log.
* **SLOWLOG**: tracks slow queries.
* **Latency monitor**: identifies performance spikes.
* Integration with Prometheus via `redis_exporter`.

---

## 18. Redis Cluster Internals

* **Cluster bus**: binary protocol on TCP port +10000 (e.g., 7000 → 17000).
* **Gossip protocol**: shares node state and slot ownership.
* **Failover**: via majority voting (similar to Sentinel).
* **Redirection commands**: `MOVED` (permanent), `ASK` (temporary).

---

## 19. Advanced Internals

* **Copy-on-write (CoW)** for forked persistence processes.
* **Memory allocator:** jemalloc, tcmalloc, or libc.
* **Fork safety:** Uses CoW pages to prevent blocking writes.
* **Lazy freeing:** Asynchronous deallocation for large keys.

---

## 20. Summary Table

| Feature         | Description                                 |
| --------------- | ------------------------------------------- |
| **Persistence** | RDB, AOF, hybrid                            |
| **Replication** | Async, partial resync                       |
| **Clustering**  | 16384 hash slots, multi-master              |
| **HA**          | Sentinel                                    |
| **Data Types**  | String, List, Set, Hash, ZSet, Stream, etc. |
| **Scripting**   | Lua                                         |
| **Modules**     | Extendable via C API                        |
| **Protocol**    | RESP2/3                                     |
| **Memory Mgmt** | Eviction, compression, CoW                  |
| **Performance** | < 1 ms latency, ~1M ops/s                   |
| **Security**    | ACL, TLS, AUTH                              |

---