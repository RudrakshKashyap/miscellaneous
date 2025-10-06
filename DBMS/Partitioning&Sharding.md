## Partitioning vs Sharding

* **Partitioning**: Dividing a large dataset (often within a single database instance) into smaller, more manageable parts called **partitions**. These partitions might be logical (such as by range, list, hash, etc.) and may or may not be stored on different physical storage/server. Conceptually, partitioning applies equally to both rowstore and columnstore.

* **Sharding**: A special case of horizontal partitioning where the data is split across multiple database instances (servers or clusters). Each “shard” holds a subset of the data. The shards together make up the full dataset. 

So, sharding = partitioning + distribution across independent instances (physically / logically separated).


* Use **partitioning** when:

  1. You’re working within one server or database engine, but have large tables that are slowing queries (e.g. time-series data, logs, etc.).
  2. You want easier maintenance (e.g. archiving old data, dropping partitions).
  3. Your transaction workloads rarely span partitions, or span only a few partitions (to reduce complexity).
  4. You don’t yet need the full cost or complexity of running many servers for data storage.

* Use **sharding** when:

  1. Data volume and load demands exceed what a single server can handle (CPU, memory, disk I/O, network).
  2. You want geographic distribution (users/data close to different regions).
  3. You need high availability and fault tolerance across server failures.
  4. You can tolerate the complexity (cross-shard consistency, routing, data balancing).

---

## Example

Assume you run an e-commerce site with orders, users, etc.

* Suppose your SQL DB has a table `orders` with 1 billion rows. Most queries involve recent orders (say last month), but you also occasionally query historical data.

  * *Partitioning option*: Partition `orders` table by `order_date` (range partitioning). Recent partitions are fast; you can drop/index/backup old ones. All partitions live in the same database server.

* As traffic and users grow, you start having performance issues: writes, reads, backups, etc. The single server can’t scale.

  * *Sharding option*: Shard by `user_id` (or geographic region). Each shard is its own database server (or cluster). Queries for a user go to that user’s shard. Less load per server. But cross-user/reporting queries need gathering from multiple shards.


| Strategy | Mechanism | Pros | Cons |
| :--- | :--- | :--- | :--- |
| **Range-Based** | Data is partitioned based on contiguous ranges of the shard key (e.g., Customer IDs 1-1000 on Shard A, 1001-2000 on Shard B). | Simple to implement; efficient for range queries. | High risk of **data skew** (hotspots) if the data or access patterns are uneven. |
| **Hash-Based** | A hash function is applied to the shard key, and the output determines the assigned shard (e.g., `hash(key) mod number_of_shards`). | Even data distribution, reducing hotspots. | Inefficient for range queries; difficult to add new shards without major data migration. |
| **Directory-Based** | A lookup table (directory) is maintained to map shard keys to specific shards. | Highly flexible; allows custom and dynamic sharding rules. | Lookup table can become a **single point of failure** and a performance bottleneck. |
| **Geo-Based** | Data is partitioned based on the geographical location of users or services. | Reduces latency for users by keeping data physically closer to them. | Can result in uneven data distribution. |

**Scalability Issues:** The whole point of sharding is to scale horizontally. An uneven distribution defeats this purpose because you can't scale by just adding new shards—the new shards won't relieve the pressure on the existing hotspot.


---

## 🧩 Context: Partitioned Table Example

Say you have a **huge table**:

```sql
CREATE TABLE orders (
    order_id BIGINT,
    order_date DATE,
    amount NUMERIC
) PARTITION BY RANGE (order_date);
```

and you create partitions:

```sql
CREATE TABLE orders_2023 PARTITION OF orders
  FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');
CREATE TABLE orders_2024 PARTITION OF orders
  FOR VALUES FROM ('2024-01-01') TO ('2024-12-31');
```

So `orders` is just a *logical table*; data physically lives in the partitions `orders_2023`, `orders_2024`, etc.

---

## 🗑️ 1. “Drop old ones” = delete whole partitions fast

If you want to remove **old data**, you don’t need to run a slow `DELETE FROM orders WHERE order_date < '2023-01-01'`.

Instead, you can instantly **drop** the entire partition:

```sql
ALTER TABLE orders DETACH PARTITION orders_2023;
DROP TABLE orders_2023;
```

This is *instantaneous* (metadata-level operation), not a row-by-row delete.
⚡ **Huge performance win** for archiving or log-style data.

---

## 🧾 2. “Index old ones” = maintain indexes separately

Each partition can have its own **index strategy**.

* Maybe recent data (2025) is queried frequently → keep multiple indexes.
* Older data (2023–2024) is rarely queried → drop some indexes to save disk and improve write speed.

Example:

```sql
-- Keep full index only on current data
CREATE INDEX idx_orders_2025_amount ON orders_2025(amount);

-- Drop older index on old partitions
DROP INDEX idx_orders_2023_amount;
```

So “index old ones” means you can independently **add or remove indexes** per partition depending on usage.

---

## 💾 3. “Backup old ones” = archive partitions independently

Because each partition is a separate physical table, you can back it up or export it individually:

```bash
pg_dump -t orders_2023 mydb > orders_2023.sql
```

Or move it to cheaper storage, or to a data warehouse (for analytics).

This avoids having to dump the *entire* `orders` table (which could be terabytes).

---

## 🔄 Queries Still “See” a Unified Table
Even though data lives in multiple child tables, you can:
```sql
SELECT COUNT(*) FROM orders;
SELECT * FROM orders ORDER BY order_date DESC LIMIT 10;
```
and the system merges results from all partitions transparently.
You don’t have to know which partition a row lives in.
When a query targets specific partitions, it uses their local indexes independently —
this improves parallelism and reduces contention.

---

| Action              | Without Partitioning                              | With Partitioning             |
| ------------------- | ------------------------------------------------- | ----------------------------- |
| **Delete old data** | `DELETE ... WHERE date < ...` (slow, locks table) | `DROP PARTITION` (instant)    |
| **Change indexes**  | Must rebuild whole table’s index                  | Manage indexes per partition  |
| **Backup/archive**  | Dump the whole table                              | Dump specific partitions only |

---
<br />
<br />
<br />




## 🗂️ How Virtual Bucket Sharding Works - A type of directory based sharding

Virtual bucket sharding introduces a two-level mapping process that decouples data from physical shards.

- **Data is mapped to virtual buckets**: A shard key (like a user ID) is processed by a hash function to determine a fixed, immutable "virtual bucket". The total number of buckets is fixed from the start and does not change.
- **Buckets are mapped to physical shards**: A separate, configurable mapping table assigns these virtual buckets to the actual physical shards in your cluster. This mapping can be changed.

### ✅ The Advantages of This Design

This two-level indirection provides key benefits for managing your database cluster:

- **Flexible Shard Management**: To add or remove a physical shard, you only need to **reassign the virtual buckets** from existing shards to the new one. The data itself already resides in the buckets; you are just changing their destination. This minimizes the amount of data that needs to be physically relocated compared to other methods.
- **Non-Contiguous Bucket Assignment**: A single physical shard can be responsible for a **non-contiguous set of virtual buckets** (e.g., buckets 1, 5, 28, and 53). This allows the **system's rebalancer** to evenly distribute buckets across shards based on their capacity or "weight," without being constrained by data ranges.


The processes for adding and removing nodes are managed by the rebalancer, triggered by configuration changes.

**Trigger Rebalancing**: The background **Rebalancer** process detects the cluster is unbalanced. It calculates a new, even distribution of buckets across all shards, including the new one.


## KEY POINTS
- Number of buckets/hash function never change, only the mapping is changed
- Ofcourse we need to store bucketId in the data(or you can create seperate files for each bucket, so you have to move only those files like what `Couchbase` does) for easier migration.

| Concept                    | **Couchbase**                | **Tarantool (vshard)**             |
| -------------------------- | ---------------------------- | ---------------------------------- |
| Bucket stored in data      | ❌ No                         | ✅ Yes (`bucket_id` field)          |
| Bucket physical separation | ✅ Yes (one file per vBucket) | ❌ No (logical field only)          |
| Rebalance                  | Move entire vBucket files    | Move tuples with given `bucket_id` |

---
<br />
<br />
<br />

# 🔄 [Consistent Hashing](https://www.youtube.com/watch?v=vccwdhfqIrI&pp=ygUSQ29uc2lzdGVudCBIYXNoaW5n): The Solution to Dynamic Sharding

Consistent Hashing is a special kind of hashing technique used primarily in distributed caching systems and distributed hash tables (DHTs) to minimize the amount of data that needs to be moved when shards (nodes) are added or removed from the system.

#### **The Problem: Why Basic Hashing Fails in Dynamic Clusters**

Consider the standard **modulo-based hashing** approach: `shard = hash(key) % N`, where `N` is the number of shards.

The critical flaw emerges when `N` changes:
*   **Adding a Shard (N+1):** The formula becomes `hash(key) % (N+1)`. This change causes *almost every key* to be remapped to a different shard. For a large database, this means nearly 100% of the data must be moved during a resharding operation, which is incredibly expensive and can make the system unavailable.
*   **Removing a Shard (N-1):** The same massive remapping problem occurs, leading to widespread data movement and potential unavailability.

This approach is brittle and doesn't work for systems that need to elastically scale in and out.

#### **The Solution: The Consistent Hashing Ring**

Consistent Hashing organizes the system as a virtual **circle (or ring)**, where the largest hash value wraps around to the smallest.

1.  **The Hash Ring:** Imagine a circle that represents the entire output range of a hash function (e.g., from 0 to 2^128 - 1).

2.  **Placing Shards on the Ring:**
    *   Each database shard (node) is assigned a unique identifier (e.g., its name or IP address).
    *   This identifier is hashed to determine the shard's position on the ring.

3.  **Placing Data on the Ring:**
    *   The key for each data item is hashed using the same hash function.
    *   This hash determines the key's position on the same ring.

4.  **Locating the Data:**
    *   To find which shard is responsible for a given key, you start at the key's position on the ring and travel **clockwise** until you encounter the first shard.
    *   This shard is the owner of that key.

    

#### **Why It's "Consistent": The Magic of Minimal Reassignment**

The core advantage is what happens when you add or remove a shard.

*   **Adding a New Shard (e.g., S4):**
    *   You hash S4's identifier and place it on the ring.
    *   Only the keys that fall between S4 and the previous shard on the ring (a subset of the keys that belonged to the next shard clockwise) need to be moved to S4.
    *   **The vast majority of keys are unaffected.**

*   **Removing a Shard (e.g., S1 fails):**
    *   When S1 is removed, the keys it owned now belong to the next shard clockwise (S2).
    *   **Only the keys that were owned by S1 need to be reassigned.** All other key assignments remain consistent.

This property is why it's called "consistent" hashing. The total number of key-shard assignments that change is minimized to `K/N`, where `K` is the total number of keys and `N` is the number of shards, rather than nearly all of them.

#### **Handling Real-World Imbalances: Virtual Nodes (Vnodes)**

![](https://towardsdatascience.com/wp-content/uploads/2024/03/17LVorrsJU4a94kyOO6UCPw-768x456.png)

A naive implementation of consistent hashing can lead to two problems:
1.  **Uneven Data Distribution:** Shards might be placed unevenly on the ring, leading to some shards owning large segments and thus more data.
2.  **Load Imbalance:** A single shard might become a hotspot if it's responsible for a very popular subset of keys.

The solution is **Virtual Nodes**.

*   Instead of placing a single point on the ring for each physical shard, each physical shard is represented by **multiple virtual nodes** scattered across the ring.
*   For example, a physical shard `S1` might be represented by virtual nodes `S1-v1`, `S1-v2`, ..., `S1-v100`, each with its own hash position.


**Benefits of Virtual Nodes:**
*   **Better Data Distribution:** Because each physical shard has many points on the ring, the segments of the ring it owns are more likely to be similar in size to others. This leads to a fairer distribution of data.
*   **Improved Load Balancing:** A popular key's load is effectively split across multiple physical shards that own the adjacent virtual nodes.
*   **Easier Rebalancing:** When a new physical shard is added, it receives a number of virtual nodes. Each virtual node inherits a small slice of data from its predecessor on the ring. This means data is migrated from *many* existing shards to the new one, preventing any single existing shard from being overloaded during the process.

## KEY POINTS
- Each shard knows exactly which hash ranges it currently owns
    - In case for SQL databases we can dynamically generate
    array of pairs `{hash("server_1_a"), "server_1"}` and do `lower_bound(hash(key))` on it
    to get where should a key belong.
- Each shard can efficiently query its local database: "SELECT * WHERE key_hash BETWEEN X AND Y"
- Modern **distributed databases**(eg- cockroachDB) often store the key_hash alongside the data for exactly this purpose. For SQL databases you would need to store shard_hash in a column.