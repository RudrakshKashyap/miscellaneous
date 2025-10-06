# [B-Tree vs. B+Tree](https://planetscale.com/blog/btrees-and-database-indexes) Page Structure (only for rowstore indexed table)

*   **[B-Tree](https://www.youtube.com/watch?v=K1a2Bk8NrYQ) (Theoretical Structure):** In a classic B-Tree, *any* node (page) can contain both keys and the actual data record (or a pointer to it). This means there is no structural distinction between an "index page" and a "data page" at the page level; they are the same thing. However, **most modern database indexes are not pure B-Trees; they are B+Trees.** A b-tree page will fit less keys, also finding all records in a range may require traversing back up and down the tree.



**Diagram: B-Tree Internal/Leaf Node Page**

```
+-------------------------------------------------------------------+
|                         B-Tree Node Page Header                   |
|-------------------------------------------------------------------|
| Page ID: 0x1A3F | Page Type: LEAF/INTERNAL | Key Count: 4        |
| Parent Page ID: 0x0C87                                           |
+-------------------------------------------------------------------+
|  Key 1: 25    |  Data/Record Ptr for 25  |  Key 2: 50   | ...    |
| (Pointer to Child Page <25) | (Pointer to Child Page 25-50) | ...|
+-------------------------------------------------------------------+
|  Key 3: 75    |  Data/Record Ptr for 75  |  Key 4: 100  | ...    |
| (Pointer to Child Page 50-75)|(Pointer to Child Page 75-100)| ...|
+-------------------------------------------------------------------+
|                       Unused Space                               |
+-------------------------------------------------------------------+
|                 Pointers to sibling pages (optional)             |
+-------------------------------------------------------------------+
```
---
<br />

*   **[B+Tree](https://planetscale.com/blog/btrees-and-database-indexes#the-btree) (Practical Implementation):** This is what databases like MySQL (InnoDB), PostgreSQL, and SQL Server use. The B+Tree has a critical design feature:
    *   **Index Pages (Non-Leaf Pages):** These pages **only contain index keys and pointers** to other pages (either lower-level index pages or leaf pages). They do not contain the actual table data.
    *   **Leaf Pages:** These pages have a very specific purpose that depends on the type of index.

So, for a **B+Tree**, yes, the index pages (the root and intermediate levels) are physically different in content and purpose from the leaf pages.



**Diagram: B+ Tree INTERNAL Node Page**

```
+-------------------------------------------------------------------+
|                    B+ Tree Internal Node Page Header              |
|-------------------------------------------------------------------|
| Page ID: 0x0C87 | Page Type: INTERNAL   | Key Count: 3           |
| Parent Page ID: 0x0001                                           |
+-------------------------------------------------------------------+
| Ptr to Child 1  |  Key 1: 50   | Ptr to Child 2  |  Key 2: 100  |
|   (Page 0x1A3F) |               |   (Page 0x2B41) |              |
+-------------------------------------------------------------------+
| Ptr to Child 3  |  Key 3: 150  | Ptr to Child 4  |               |
|   (Page 0x3C15) |               |   (Page 0x4D29) |  (Unused)    |
+-------------------------------------------------------------------+
|                           Unused Space                           |
+-------------------------------------------------------------------+
```

* **Node Size** - The key idea is to size **one B+ tree node to fit exactly into one database page**. When the database needs to read a node (e.g., during a search), it reads the entire page containing that node from disk in a single I/O operation. By packing as many keys as possible into a single node/page, the **tree's fan-out (branching factor) increases**, which **decreases the tree's height**. A shorter tree means fewer disk I/Os are needed to traverse from the root to a leaf, dramatically speeding up queries.

    For example, if a node sized to a 16KB page can hold 682 keys, a tree with just 3 levels can store over 300 million keys (`682 * 682 * 682`), and finding any key requires reading only 3 pages from disk.

---
<br />
<br />
<br />

# [LSM Tree](https://www.youtube.com/watch?v=I6jB0nM9SKU&list=PLvNwhCZmi4xzFftMjSDhnAEKzG4KIebUf&index=29)

![](https://miro.medium.com/v2/resize:fit:720/format:webp/0*JeaDeiI9ncQMrqGK.png)

The **Log-Structured Merge-Tree (LSM Tree)** is a disk-based data structure designed to provide low-cost indexing for files experiencing a high rate of writes (inserts, updates, deletes). It was first described in a 1996 paper by Patrick O'Neil.

The core idea is to **sacrifice some read performance for a massive gain in write performance** by transforming random writes into sequential writes.

#### Key Components of an LSM Tree System:

1.  **In-Memory Buffer (MemTable)**
2.  **Write-Ahead Log (WAL)**
3.  **Immutable Sorted String Tables (SSTables)**
4.  **Background Compaction Process**

Let's see how these components work together.

---

### Part 3: The Internal Workflow of an LSM Tree

#### Step 1: The Write Path

1.  **Write to WAL:** When a write (insert, update, delete) comes in, it is first appended to a **Write-Ahead Log (WAL)** file on disk. This is a sequential write and is very fast. The purpose of the WAL is durability; it allows the database to recover in-memory data that was lost in case of a crash.

2.  **Update the MemTable:** After being logged to the WAL, the data is inserted into an in-memory balanced tree structure (like a **Red-Black Tree** or a Skip List) called the **MemTable**. This structure keeps the data sorted by key. Writes to memory are extremely fast.

#### Step 2: Flushing to Disk - Creating an SSTable

3.  **Reaching a Threshold:** When the MemTable reaches a certain size (e.g., a few hundred MB), it is marked as **immutable** (read-only). A new, empty MemTable is created to handle incoming writes.

4.  **Flushing to SSTable:** The immutable MemTable is then flushed to disk as a file called an **SSTable (Sorted String Table)**. Since the MemTable was already sorted, writing it out is a **sequential I/O operation**, which is much faster than random I/O.

**This is the first major performance win: transforming random writes into sequential writes.**

---

### Part 4: What is an [SSTable](https://www.youtube.com/watch?v=b_I2somTxuw)?

An **SSTable (Sorted String Table)** is a simple, persistent file format that is fundamental to LSM Trees.

**Key Properties of an SSTable:**
*   **Immutable:** Once written, it is never modified. This simplifies caching and concurrency control.
*   **Sorted by Key:** All key-value pairs within the file are sorted by key. This is crucial for efficient reading and merging.
*   **Divided into Blocks:** The file is typically divided into data blocks (e.g., 4KB-64KB) to facilitate reading and caching.
*   **Includes Indexes:** An SSTable file usually has:
    *   A **data section** (the sorted key-value pairs).
    *   A **sparse index** that stores the key and the offset for the start of every Nth data block.
    *   A **footer** that points to the index.

**How a Lookup in a Single SSTable Works:**

![](https://miro.medium.com/v2/resize:fit:720/format:webp/0*yNh-t8-X5XRIeprg.png)

To find a key in an SSTable, the system doesn't scan the whole file.
1.  It performs a binary search on the sparse index(typically cached in memory) to find the correct data block.
2.  It then seeks to that block on disk and reads it into memory.
3.  Finally, it does a quick scan within that block (which is now in memory) to find the specific key-value pair.

Over time, you have multiple SSTables on disk (from multiple MemTable flushes). Each SSTable contains data that is sorted internally, but different SSTables can have overlapping key ranges.

---

### Part 5: The Read Path and the "Problem" of Multiple SSTables

To read a value for a given key, the system must:

1.  Check the active MemTable.
2.  Check the immutable MemTable (if it exists).
3.  Check the SSTables on disk, typically from the **newest to the oldest**.

Why newest to oldest? Because a newer SSTable will contain a more recent value for a key that exists in multiple SSTables (updates are just new writes).

**The Challenge:** If you have many SSTables, a read could require checking many files, which is slow. This is the primary **trade-off** of LSM Trees: writes are blazingly fast, but reads can become slower as the number of SSTables grows.

**Solutions to Optimize Reads:**
*   **[Bloom Filters](https://youtu.be/V3pzxngeLqw?si=Fi6Htk9eGwlmtQiA&t=78):** A probabilistic data structure that can tell you with 100% certainty if a key is *not* in an SSTable, and with high probability if it *might* be. Before checking an SSTable, the database checks its Bloom filter. If the filter says "no," it skips that file entirely. This drastically reduces the number of files to check.
*   **Caching:** Frequently accessed data blocks and index blocks are cached in memory.

In LSM tree implementations, **each SSTable has its own Bloom filter.**
```python
# Example lifecycle
SSTable1 (1MB data) → BloomFilter1 (sized for 10,000 keys)
SSTable2 (1MB data) → BloomFilter2 (sized for 10,000 keys) 
During compaction: SSTable1 + SSTable2 → SSTable3
Result: Delete SSTable1, SSTable2, BloomFilter1, BloomFilter2
        Create SSTable3 with new BloomFilter3
```
---

### Part 6: The Secret Sauce - Compaction

To solve the problem of too many SSTables and to reclaim space from overwritten or deleted data, LSM Trees use a process called **Compaction**.

**What is Compaction?**
Compaction is a background process that merges multiple SSTables into a new, single, larger SSTable.

**What happens during merging?**
*   Since all input SSTables are sorted, the merge can be done efficiently in a single pass (like the merge phase of a merge-sort).
*   When multiple versions of the same key are found, only the **most recent** version is kept in the new SSTable. Old versions are discarded.
*   **For deleted keys (which are marked with a "tombstone" marker during a delete operation), the TOMBSTONE and all previous versions are discarded once the compaction determines the deletion is final.**

**The Result of Compaction:**
*   The number of SSTables is reduced.
*   Data is consolidated, making reads faster.
*   Disk space is freed up.

There are different compaction strategies (e.g., **Size-Tiered** in Cassandra, **Leveled** in LevelDB/RocksDB), but the core principle of merging sorted files remains the same.

---

### Summary: The Complete Picture

| Aspect | How it Works in an LSM Tree | Why it's Efficient |
| :--- | :--- | :--- |
| **Write** | 1. Append to WAL (sequential).<br>2. Write to in-memory MemTable. | Transforms random writes into sequential I/O. Extremely fast. |
| **Flush** | Convert full MemTable to an immutable, sorted SSTable file on disk. | Sequential disk write. Creates sorted files for efficient future reads. |
| **Read** | 1. Check MemTables.<br>2. Check SSTables (newest to oldest) using Bloom Filters and indexes. | Bloom filters prevent unnecessary disk lookups. Sparse indexes minimize data read. |
| **Cleanup** | Background compaction merges SSTables, removes duplicates, and applies deletions. | Maintains read performance and reclaims disk space over time. |

### Popular Databases Using LSM Trees

*   **Apache Cassandra:** Uses a variant of LSM Trees with Size-Tiered Compaction.
*   **HBase:** Uses LSM Trees on top of HDFS.
*   **Google Bigtable:** The original, which inspired HBase.
*   **RocksDB (by Facebook):** A highly optimized embeddable KV store using Leveled Compaction. It's used as the storage engine for **MySQL MyRocks**, **MongoDB's WiredTiger** (with some modifications), and many other systems.
*   **LevelDB (by Google):** The predecessor to RocksDB.
*   **InfluxDB:** A time-series database that uses LSM Trees.