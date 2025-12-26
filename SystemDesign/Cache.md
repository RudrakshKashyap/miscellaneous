# Caching strategies

* **Definition**: Temporarily storing copies of data (or computation results, or instructions) in a faster storage layer, so that future requests for that data can be served more quickly.
* **Purpose**: Reduce latency, reduce load on slower systems (disk, network, DB), improve throughput, reduce cost (because fast memory / fast paths are expensive / limited).
* **Cache levels / layers**: hardware cache (CPU L1/L2/L3), OS‐disk cache, in-memory cache (e.g. Redis, Memcached), distributed caches, CDN, browser cache, etc.

---

## Key Concepts & Metrics

* **Hit vs Miss**: A “hit” is when the requested data is found in the cache; “miss” is when it isn’t, so you fetch from the slower underlying store.
* **Latency & Throughput**: Caches reduce latency (faster fetches), increase throughput by reducing work on backends.
* **Capacity**: Size of cache (in entries, bytes, etc.) constrains what can be held.
* **Consistency / Freshness / Staleness**: How up-to-date the cache content is relative to the source.
* **Eviction / Replacement**: When cache is full, deciding what to remove.
* **Write policies**: For handling writes (when data changes) — ensuring the cache and the source are consistent.
* **Read strategies**: When and how data is brought into cache.
* **TTL / Expiry**: Time-based eviction / invalidation when data becomes stale after some duration.

---

<img src="https://substackcdn.com/image/fetch/$s_!DUW9!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F3d4f861a-82b2-4b8f-b9c7-d926f079a108_2163x3153.jpeg" width="40%" height="auto" />


## Read Strategies

These govern how reads are handled with respect to the cache and backend:

| Strategy                       | How it works                                                                                                                                              | Pros                                                                                                                   | Cons                                                                                                                                                                 |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cache-Aside (Lazy Loading)** | Application checks cache. If hit, return. If miss, fetch from backend, put into cache, return.                                                            | Simpler, fine control. Can avoid caching unwanted data. Good when reads are frequent and data access patterns diverse. | Application complexity: you need to handle cache misses. Risk stale data if invalidation not handled. Also more code to manage consistency. ([backendgarden.com][1]) |
| **Read-Through**               | The cache itself is an intermediary: if the item is not in cache, the cache fetches from backend, stores it, returns it. Client just reads through cache. | Transparent to client; easier to maintain; good read performance.                                                      | Slight overhead in cache manager; may be more complex to implement; possible cache misses still cause backend load. ([systemsarchitect.io][2])                       |

---

## Write Strategies

When data changes (writes, updates), how the cache + backend stay consistent. Main ones:

| Writing Strategy                                                       | How it works                                                                                                                       | Pros                                                                                                                                                  | Cons                                                                                                                                                                                        |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Write-Through**                                                      | On write, you write to both cache and backend synchronously. Cache always has fresh data.                                          | Strong consistency; simplifies read logic (you don’t usually hit stale data).                                                                         | Slower write performance (because backend write needed every time). Higher latency. More load on backend. ([systemsarchitect.io][2])                                                        |
| **Write-Behind** (also called **Write-Back**, or “asynchronous write”) | Write to cache first; backend is updated later (asynchronously / in batch).                                                        | Very fast writes; reduces load/spikes on backend. Suitable when write latency critical.                                                               | Risk of data loss if cache fails before backend sync; more complex consistency / recovery logic. Possible stale data. ([backendgarden.com][1])                                              |
| **Write-Around**                                                       | Writes go directly to the backend, bypassing the cache. Reads still check cache first. The cache gets populated when data is read. | Reduces cache pollution (i.e. avoids caching data that might be written but never read). Good if writes are common and reads less likely immediately. | Read misses more likely after writes (because writes don’t update cache). Could cause higher read latency. Cache might not reflect latest data until after a read. ([backendgarden.com][1]) |

---

## Eviction / Replacement Policies

Because cache size is finite, decisions must be made to evict (remove) existing entries to make room for new ones. There are many policies; choosing depends on access patterns, workloads, etc.

Some common eviction (replacement) policies:

| Policy                                | How it works                                                                                                                  | Best use-cases                                                                                                                                 | Trade-offs                                                                                                                                                                                           |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LRU (Least Recently Used)**         | Evict the item which was used least recently. Assumes items used recently will be used again.                                 | Good when recency is a strong predictor (e.g. user sessions, web caches).                                                                      | Need to track access order (metadata / overhead). May evict items that are infrequently used but might be still important long-term.                                                                 |
| **LFU (Least Frequently Used)**       | Evict the item accessed least often over time.                                                                                | When frequency matters more than recency; stable “hot” items should stay.                                                                      | Tracking counts adds overhead; “cold start” problem (new items have low counts); old popular items that aren’t used currently might still stick around; adaptiveness to changing patterns is harder. |
| **FIFO (First In, First Out)**        | Evict the item that has been in the cache the longest (oldest insertion) regardless of access.                                | Simple workloads; low overhead; when recency/frequency are less useful.                                                                        | May evict items that are heavily used recently; not sensitive to usage; likely lower hit rates in many realistic workloads.                                                                          |
| **MRU (Most Recently Used)**          | Evict the most recently used item first. (Opposite assumption to LRU.)                                                        | Rare, but useful in specific workloads, e.g. when items once used are unlikely to be used again soon (certain streaming / scanning workloads). | Usually counterintuitive; bad for many general workloads.                                                                                                                                            |
| **Random Replacement**                | Pick a random item to evict.                                                                                                  | Very low implementation overhead; useful if tracking is too expensive.                                                                         | Poor predictability; likely worse hit rates. Somewhat unpredictable performance.                                                                                                                     |
| **Hybrid Policies / Adaptive Caches** | Combine or adapt between recency and frequency (for example, segmented LRU, ARC (Adaptive Replacement Cache), TinyLFU, etc.). | When access patterns are dynamic; you want the best of multiple policies; when workloads change over time.                                     | More complex to implement; more metadata; possibly more expensive in CPU/memory. ([ByteByteGo][3])                                                                                                   |

Also:

* **TTL (Time to Live) / expiry**: each cache entry has a time after which it is considered stale or invalid; even if “hot,” after TTL; helps with ensuring freshness. Some systems combine TTL with eviction. ([deepaksood619.github.io][4])
* **Size-based eviction**: if cache has a size limit (in bytes or memory), must evict based on size. Eviction policies often have to consider entry sizes.


| Method                                             | How it works                                                                                                                                                                                                                                                    | Pros                                                                                                                                    | Cons / When it becomes difficult                                                                                                                                                                      |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Time-based Expiration / TTL (Time-to-Live)**     | Each cache entry is given a TTL; after that time it is considered stale and either removed or refreshed. ([GeeksforGeeks][1])                                                                                                                                   | Simple to implement; no need for external triggers; predictable overhead; works well for data that changes at somewhat known intervals. | TTL too long → stale data; TTL too short → lots of refreshes, losing benefit of cache; unpredictable update times (if data changes in bursts) make setting TTL hard.                                  |
| **Manual (or Explicit) Invalidation**              | When the underlying data changes, the application / service code explicitly tells the cache which keys to invalidate, or issues a purge or refresh. ([GeeksforGeeks][1])                                                                                        | Precise control; you invalidate only what changed; good for critical data that cannot be stale.                                         | Requires you to correctly track dependencies (which cache keys correspond to what data change). More coding. Possibility of forgetting to invalidate (bugs).                                          |
| **Event-Driven / Notification-Based Invalidation** | When a data change (insert / update / delete) occurs, an event / message is emitted (e.g. via pub/sub, message queue) that informs cache(s) to invalidate relevant entries. ([MoldStud][2])                                                                     | More automatic; can be near real-time; works well in distributed env; better consistency.                                               | Complexity: need infrastructure for streaming / messaging; needs reliable delivery; handling ordering / concurrency; determining which keys to invalidate; potential performance cost if many events. |

---

## Combined / Different Strategies & Patterns

Here are patterns that combine read/write/eviction strategies:

* **Cache Aside + Write-Around**: Application controls cache; writes bypass cache; only reads populate it. Good when writes are many but reads less frequent.
* **Read-Through + Write-Through**: Client always goes through cache; writes are synchronous to both; cache is always consistent. Good for strong consistency requirements.
* **Read-Through + Write-Back**: Client reads through cache; writes go to cache and then asynchronously to backend. Speeds up writes; some risk of inconsistencies.
* **Write-Back + Eviction policies with persistence / logging**: To reduce data loss risks, many write-behind systems use journaling / logging of writes (so that if cache crashes, can recover).

---

## Trade-Offs & Factors to Consider

When picking caching / eviction / write/read strategy, many trade-offs. Here are factors to consider:

1. **Latency Requirements**
   How real-time is the data? Can you tolerate slightly stale data? If not, strategies like write-through and short TTLs are needed.

2. **Consistency Requirements**
   Does the application require strong consistency(Write-Through) between cache and backend? Or eventual consistency(Write-Behind, TTL) is fine?

3. **Read vs Write Ratio**
   If reads dominate, optimizing for reads (read-through, cache-aside) and good eviction = high hit rate. If writes dominate, write performance, avoiding cache pollution, dealing with write load matter more.

4. **Failure / Durability Risks**
   Especially with write-behind, you risk data loss if cache fails. Need mechanisms to mitigate (persistent cache, replication, logging, etc.).

5. **Cache Size / Memory Constraints**
   Overhead of metadata (for LRU, LFU etc.) matters. Very large caches might make some policies inefficient in time/space.

6. **Access Pattern Characteristics**

   * Are there a few “hot” items accessed frequently and many “cold” ones?
   * Are patterns changing over time (temporal locality)?
   * Are accesses bursty?
   * Is there scanning (iterating through many items once)?

7. **Cost (Complexity, Infrastructure, Development)**
   More sophisticated strategies give potential benefits but cost more to implement, debug, maintain.

8. **Distributed vs Local Cache**
   If the cache is local (single node), simpler; distributed cache introduces network latency, consistency issues, invalidation overhead.

9. **Eviction Overhead**
   How expensive is it to maintain the policy? E.g. LRU: update access time; LFU: update counters; more complex hybrids need more bookkeeping.

---

## When to Use Which Strategy: Examples

Here are some example scenarios & what strategies tend to work well:

| Scenario                                                                 | Likely Good Strategy / Policies                                                                                                                                                                  |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Web pages / CDN for mostly static content, but with updates occasionally | Read-through or cache aside for read; TTL‐based invalidation; eviction policy: LRU or something simple; maybe write-around or write-through depending on how important immediate consistency is. |
| Session data / user profiles                                             | Need consistency; write‐through or write‐behind with safety; eviction policy LRU; ensure TTL for sessions; possibly cache aside if backend can handle.                                           |
| Log systems or analytics: many writes, fewer reads immediately           | Write-back, write-around; maybe lazy loading; eviction might be based on age or size; freshness less critical if data used for batch jobs.                                                       |
| Highly dynamic data / auctions / stock prices                            | Strong consistency required; probably write-through; small TTLs; aggressive invalidation; eviction policy tuned for recency.                                                                     |
| Mobile / edge / browser caching                                          | Need to minimize network calls; read-through or cache aside; possibly offline fallback; eviction by recency/frequency; TTL critical.                                                             |

---


# Caching Pitfalls - TODO
https://www.youtube.com/watch?v=wh98s0XhMmQ