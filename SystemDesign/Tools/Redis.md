# [**Redis**](https://www.mybluelinux.com/redis-explained/)

Check out above link, very well explained

![](https://www.mybluelinux.com/img/post/posts/0189/redis-explained-infographic.webp)

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
[`fork()`](../../OS/FDs&IOchannels.md#file-access) → child writes a compact AOF while parent appends new commands to buffer → merged at the end.

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

* **Redis Sentinel** is for monitoring and failover of *standalone master–replica* deployments. Sentinels (external) coordinate failover.

* **Redis Cluster** is the sharded deployment: it uses a **gossip protocol** and an internal election/voting among masters for failover (masters vote), not Sentinels. Both systems use voting/quorum, but the actors differ (Sentinels vs cluster masters). Clarify that difference.


---

## 8. Clustering (Virtual Bucket Sharding)

Redis Cluster enables **horizontal scaling** via **sharding and replication**.

### **Architecture:**

* **16384 hash slots** distributed across masters.
* Each key’s slot = `CRC16(key) % 16384`.
* Automatic data rebalancing.
* Redirects using `MOVED`/`ASK` responses.

### **Limitations:**

* Multi-key operations only allowed within same slot.
* Some commands restricted.

```pgsql
               ┌─────────────────────────┐
               │       Redis Cluster      │
               └─────────────────────────┘

                (16384 Hash Slots Distributed)

          ┌──────────┐         ┌──────────┐         ┌──────────┐
          │ Master A  │        │ Master B  │        │ Master C  │
          │Slots 0-5460│       │Slots 5461-10922│  │Slots 10923-16383│
          └─────┬─────┘        └─────┬─────┘        └─────┬─────┘
                │                   │                      │
         ┌──────┴──────┐       ┌──────┴──────┐       ┌──────┴──────┐
         │ Replica A1   │      │ Replica B1   │      │ Replica C1   │
         └──────────────┘      └──────────────┘      └──────────────┘
         ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
         │ Replica A2   │      │ Replica B2   │      │ Replica C2   │
         └──────────────┘      └──────────────┘      └──────────────┘
```

To store related keys together, Redis supports:
```pgsql
user:{123}:name
user:{123}:age
```
Only the part inside `{ }` is hashed.
So all keys with `{123}` land on the same shard.

Then you can get all keys at once:

```
MGET user:{100}:name user:{100}:age user:{100}:email
```


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

### [**Pub/Sub**](https://youtu.be/fmT5nlEkl3U?si=iSuKKCUgiyovnOWO&t=1600)

* Real-time message broadcasting.
* Non-persistent.
* Commands: `PUBLISH`, `SUBSCRIBE`, `PSUBSCRIBE`, etc.
* UseCase - CHECK ABOVE VIDEO

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


<br />
<br />
<br />

# MORE INFO

This is a critical two-step process in Redis High Availability. The system first needs to agree on **who** runs the show (the Leader Sentinel), and then that Leader must decide **which** database node is best suited to take over (the new Master).

---

### 1. The Election: How a "Leader Sentinel" is Selected

When a Sentinel realizes the Master is objectively down (ODOWN), it doesn't just immediately trigger a failover. It must first win an election among the other Sentinels to become the authorized "Leader."

This uses the Raft consensus algorithm logic.

**The Election Steps:**

1.  **Epoch Increment:** The Sentinel increments its current `configuration epoch` (essentially a counter for the current "term" or "round" of voting). Each sentinel may vote only once per epoch.
2.  **Request for Votes:** The candidate Sentinel sends a command (`SENTINEL is-master-down-by-addr`) to all other active Sentinels, essentially asking: *"The Master is down. If you haven't voted for anyone else in this epoch, vote for me to perform the failover."*
3.  **First-Come, First-Served:** A Sentinel will vote for the **first** candidate that asks it for a vote within a specific epoch. It cannot change its vote once cast for that epoch.
4.  **Winning the Election:** To become the Leader, a Sentinel must receive "Yes" votes from a **majority** of the Sentinels (specifically, more than 50% of the total configured Sentinels).
    * *Note:* This is stricter than the "Quorum" setting. Even if Quorum is 2, but you have 5 Sentinels, you need 3 votes to become Leader.



> **Why this matters:** This prevents "Split Brain" scenarios where two different Sentinels try to promote two different Replicas to Master at the same time.

---

### 2. The Promotion: How the "New Master" is Selected

Once a Leader Sentinel is elected, it looks at the available Replicas (slaves) to decide which one should be promoted. It does not pick randomly; it uses a strict four-step filtering and ranking process.

The Leader Sentinel evaluates the Replicas in the following order:

#### Step A: Filter out the "Bad" Candidates
Before ranking, the Leader removes any Replica that is ineligible. A Replica is disqualified if:
* It is currently disconnected from the Sentinel.
* It has been disconnected from the old Master for too long (specifically, longer than `(down-after-milliseconds * 10) + time_since_master_down`).

#### Step B: Rank the Remaining Candidates
The Leader compares the remaining eligible Replicas using these criteria, in this exact order:

**1. Replica Priority (User Configured)**
* Redis checks the `replica-priority` setting in `redis.conf`.
* **Lower is better.** A replica with priority 10 is chosen over one with priority 100.
* *Note:* If a replica has a priority of `0`, it is **never** elected.

**2. Replication Offset (Data Freshness)**
* If priorities are equal, the Sentinel checks the replication offset.
* The offset represents how much data the Replica has received from the Master.
* **Higher is better.** The Replica with the highest offset has the most up-to-date data.

**3. Run ID (Tie Breaker)**
* If both Priority and Offset are identical, Redis compares the `Run ID` (a random string generated when a Redis node starts).
* The Replica with the **lexicographically smaller** Run ID is chosen.
* *Note:* This is purely arbitrary. It ensures that if two Replicas are identical in every way, the Sentinels deterministically pick the same one every time.



### Summary of the Workflow

| Stage | Goal | Mechanism | Winner |
| :--- | :--- | :--- | :--- |
| **Stage 1** | **Pick a Controller** | Raft-like Voting | The Sentinel that gets > 50% of the votes. |
| **Stage 2** | **Pick a Database** | Filtering & Ranking | The Replica with the best Priority $\rightarrow$ most Data. |


## In case of redis cluster
all nodes uses gossip protocol to check the health of cluster and 
determine if master is offline. The replica itself initaiates the election and master ntdes vote for the replica.

---

<br />
<br />
<br />

# COMPARISION
This detailed breakdown explains the architectural and mechanical differences between Redis, Raft, Paxos, and Lease-based systems.

### 1. Redis Replication & Failover vs. Raft

The fundamental difference lies in their design philosophy: **Redis prioritizes availability and latency**, whereas **Raft prioritizes strong consistency (CP in CAP theorem).**

#### A. Replication Mechanism
* **Redis (Asynchronous Replication):**
    * **Data Path:** When a client writes to the Master, the Master writes to its memory, acknowledges the client immediately, and *then* asynchronously sends the command to Replicas.
    * **Consequence:** If the Master crashes *after* acknowledging the client but *before* sending the command to the replica, **that write is lost forever**. Redis does not wait for replicas to confirm they received the data.
    * **Log:** Redis uses an append-only file (AOF) or snapshot (RDB) for persistence, but the replication stream is just a stream of commands, not a strictly ordered, shared consensus log.

* **Raft (Consensus-based Replication):**
    * **Data Path:** When a client writes to the Leader, the Leader appends the entry to its log but **does not apply it yet**. It replicates this entry to Followers.
    * **Commit:** The Leader waits until a **majority (quorum)** of nodes have written the entry to their logs. Only then does it "commit" the entry, apply it to its state machine, and acknowledge the client.
    * **Consequence:** Once a client receives a success response, the data is guaranteed to be durable even if the leader crashes.

#### B. Failover & Leader Election
* **Redis (Sentinel):**
    * **Separation of Concerns:** Redis separates the *data nodes* (Master/Replica) from the *monitoring nodes* (Sentinels).
    * **The Process:**
        1.  Sentinels monitor the Redis Master.
        2.  If the Master becomes unresponsive, Sentinels vote *among themselves* (using a Raft-like algorithm) to elect a "Leader Sentinel."
        3.  The **Leader Sentinel** then picks a Redis Replica and promotes it to Master.
    * **Split-Brain (The "Last Failover Wins" Problem):** If a network partition occurs, the old Master might still be accepting writes from clients on its side of the partition. Meanwhile, Sentinel promotes a new Master on the other side. When the partition heals, the old Master is demoted, and **all writes it accepted during the split are wiped out** to match the new Master.

* **Raft (Integrated Election):**
    * **Integrated System:** Every node participates in both data storage and leader election. There are no external "watchers."
    * **The Process:** If a Follower hears nothing from the Leader (timeout), it becomes a Candidate and requests votes. If it gets a majority of votes, it becomes the new Leader.
    * **Split-Brain Safety:** Raft uses **Terms** (logical clocks). If an old Leader tries to replicate logs, followers will reject them because they see a higher Term from the new Leader. Crucially, the old leader cannot "commit" any new writes during a split because it cannot reach a majority.

| Feature | Redis (Sentinel) | Raft |
| :--- | :--- | :--- |
| **Write Consistency** | Eventual (Async) | Strong (Linearizable) |
| **Data Loss Risk** | Yes (during failover/partition) | No (committed entries are safe) |
| **Failover Arbiter** | External processes (Sentinels) | Internal nodes (Self-electing) |
| **Split Brain** | Old master accepts writes (data loss) | Old master cannot commit writes (stalls) |

---

### 2. Paxos & Multi-Paxos

Paxos is the academic ancestor of Raft. It is a protocol to reach consensus on a **single value** among a network of unreliable processors.

#### Basic Paxos (Single Decree)
Paxos works in two phases to agree on one single value (e.g., "Who is the leader?").
1.  **Phase 1 (Prepare/Promise):**
    * A **Proposer** selects a proposal number $N$ and sends a `PREPARE(N)` message to a quorum of Acceptors.
    * **Acceptors** respond: "I promise not to accept any proposal numbered less than $N$. Also, here is the highest numbered proposal I have already accepted (if any)."
2.  **Phase 2 (Accept/Accepted):**
    * If the Proposer gets promises from a majority, it sends `ACCEPT(N, Value)` to the Acceptors.
    * **Acceptors** accept the value unless they have already promised to a higher $N$.
    * Once a majority accepts, the value is **Learned**.

#### Multi-Paxos
Basic Paxos is slow because it requires 2 round trips (Phase 1 + Phase 2) for *every single piece of data*. Multi-Paxos optimizes this for a **stream of values** (like a database log).
* **The Optimization:** Instead of running Phase 1 for every entry, the nodes elect a **Stable Leader**.
* **Skipping Phase 1:** Once a Leader is established (Phase 1 complete), it can run **only Phase 2** for subsequent log entries.
* **Benefit:** This reduces latency to 1 round trip, making it as fast as Raft. (Raft is essentially a more understandable, constrained version of Multi-Paxos).

---

### 3. Lease-Based Election

Lease-based election is a method often used when you want a simple leader without the complexity of a full consensus algorithm like Paxos/Raft for every decision. It relies on **time** rather than just message passing.

#### How it works
1.  **Acquire:** Nodes compete to write a key to a shared store (like a database or ZooKeeper) with a "Time-to-Live" (TTL), e.g., 10 seconds.
2.  **Win:** The first node to successfully write the key becomes the Leader.
3.  **Renew:** The Leader must periodically "heartbeat" (update the TTL) to keep the lease.
4.  **Expire:** If the Leader crashes, it stops renewing. After 10 seconds, the key expires. Other nodes see the key is gone and try to acquire it again.

#### Critical Differences & Risks
* **Clock Drift:** This method assumes the clocks on the server and the client move at roughly the same speed. If the Leader's clock is slow, it might think it still holds the lease, while the shared store thinks it expired and gave it to someone else.
* **Fencing Tokens:** To solve the above, systems use a **Fencing Token** (a monotonic number).
    * When a leader acquires a lease, it gets token `3`.
    * If it freezes and wakes up later thinking it's still the leader, it tries to write with token `3`.
    * Meanwhile, a new leader has acquired the lease with token `4`.
    * The storage system sees `3 < 4` and rejects the old leader's request.

### Summary Table

| Concept | Core Mechanism | Best Used For |
| :--- | :--- | :--- |
| **Redis Sentinel** | External watchers + Async Replication | Caching, High Speed, tolerance for minor data loss. |
| **Raft** | Log replication + Majority Vote | Distributed Databases (etcd, TiKV), Strong Consistency. |
| **Paxos** | Proposer/Acceptor phases | The theoretical foundation of consensus (hard to implement). |
| **Multi-Paxos** | Stable Leader optimization of Paxos | Practical implementation of Paxos for logs (Spanner). |
| **Lease-Based** | Time-bound locks (TTL) | Simple leader election (e.g., Kubernetes controller manager). |


<br />
<br />
<br />

# Basically wait is useless it just is letting client know that the change is not replicated successfully to all replicas

Redis offers the WAIT command (client can block until N replicas have acknowledged a write) and server configs like min-replicas-to-write / min-replicas-max-lag. These make writes stronger (less likely to be lost) but are still not the same as Raft’s commit-before-ack semantics at the protocol level. Mention these so readers know Redis can be tuned for durability at some cost to latency.


This is the exact reason why Redis is often criticized when people try to use it as a "durable" database rather than a cache.

If `WAIT` times out and N replicas have **not** acknowledged:

1.  **The write stays on the Master.** It is **not** rolled back.
2.  **`WAIT` returns a number smaller than N.** (e.g., if you asked for 2 and got 1, it returns `1`).
3.  **The client gets control back.**

Your intuition is correct: **The database is now in a dangerous state.** The Master has data that the Replicas do not have. If the Master crashes right now, that data is lost forever, even though the Master accepted it.

### So, what is the point of blocking?

If the write isn't guaranteed, why bother blocking the client? The value isn't in **guaranteeing** safety, but in **exposing** danger.

#### 1. It gives the Client a choice (The "Application-Level" Rollback)
Without `WAIT`, your application fires a write and blindly assumes it's safe.
With `WAIT`, if you receive an acknowledgment of `0` or `1` (when you wanted `2`), your application code knows **"This write is risky."**

You can then write code to handle this:
* **Retry:** Try the write again?
* **Alert:** Log a critical error that the cluster is degraded?
* **Compensate:** Issue a `DEL` command to manually undo the write you just made (a manual rollback).

#### 2. It creates "Backpressure" (Flow Control)
This is the most practical use case. If your Replicas are failing or the network is slow, `WAIT` forces your writing clients to slow down.
* **Without `WAIT`:** The Master accepts 100,000 writes/sec, filling up its memory. The Replicas are dead, so they get none of it. When the Master eventually crashes, you lose **huge** amounts of data.
* **With `WAIT`:** The first write blocks. The client stops sending new data until the Replicas catch up or the timeout occurs. You lose **one** write instead of 100,000.

#### 3. It serves as a "Check" for Reads
If you use `WAIT` successfully, you gain a property called **Read-your-writes** consistency (mostly).
If `WAIT` returns `2` (and you have 2 replicas), you know that if you immediately disconnect and connect to a Replica to perform a `GET`, that Replica *definitely* has the data you just wrote.


The point of `WAIT` is not to guarantee the database is correct (it can't), but to **tell the client** that the database might be incorrect. It shifts the responsibility of consistency from the **Database** (where it belongs in Raft) to the **Application Developer** (you).