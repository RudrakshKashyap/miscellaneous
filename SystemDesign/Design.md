
# [Microservices](https://www.youtube.com/watch?v=rv4LlmLmVWk)

The best choice depends heavily on your project's scale, team structure, and specific challenges. For many projects, starting with a modular monolith is a highly recommended strategy.<br />
A well-designed modular monolith is an excellent stepping stone. It allows you to build a system with clear boundaries that can be **split into microservices later if truly needed**.

| Feature | Modular Monolith | [Microservices](https://www.youtube.com/watch?v=lTAcCNbJ7KE) |
| :--- | :--- | :--- |
| **Architecture & Deployment** | Single codebase and deployment unit with isolated modules | Multiple, independently deployable services |
| **Development & Debugging** | Simplified due to a single codebase; easier to trace and test | More complex; requires tracing across network calls |
| **Communication** | In-process method calls; faster and more reliable | Network calls (APIs); introduces latency and complexity |
| **Data Management** | Simpler; modules can share a single database, easing transactions | Complex; each service often has its own database, eventual consistency |
| **Scalability** | Scale the entire application as a unit; less granular | Scale individual services independently for efficiency |
| **Technology Stack** | Typically a single, unified technology stack | Freedom to use different technologies per service |
| **Operational Complexity** | Lower; one application to deploy and monitor | Higher; requires orchestration, service discovery, and distributed monitoring |
| **Cost** | More cost-effective initially due to simpler infrastructure | Higher infrastructure and operational costs |



# [Event-Driven Architecture](https://www.youtube.com/watch?v=hrvx8Nv9eQA)

**Event-Driven Architecture (EDA)** is a design pattern where a system is built to produce, detect, consume, and react to **events**.

---

### Core Concepts

1.  **Event Producer (Publisher):** The service that detects or creates an event and publishes it to a channel.
2.  **Event Router (Channel/Broker):** The middleware that accepts events from producers and routes them to the appropriate consumers. Common technologies are **Apache Kafka**, **RabbitMQ**, or **AWS EventBridge**.
3.  **Event Consumer (Subscriber):** The service that listens for events and takes action in response.

### Key Communication Styles

*   **Pub/Sub (Publish-Subscribe):** An event is broadcast to *all* consumers who have subscribed to that event type. The producer doesn't know who the consumers are. Event is gone once consumed, we dont keep it.
    
*   **Event Streaming:** Events are written to a log (a stream). Consumers can read from this log at their own pace and are not limited to just new events; they can replay past events.

### Benefits

*   **Loose Coupling:** Producers and consumers are independent and don't need to know about each other. This makes the system more modular and easier to change.
*   **Asynchronicity:** Producers don't have to wait for consumers to process the event, leading to more responsive systems.
*   **High Scalability:** The event router can handle a massive flow of events, and consumers can be scaled independently.
*   **Resilience:** If a consumer fails, events can be stored in the broker and processed when the consumer recovers.

### Challenges

*   **Complexity:** Debugging and tracing a flow of events across multiple services can be difficult.
*   **Eventual Consistency:** The system is not immediately consistent everywhere, as it takes time for all consumers to process the event.
*   **Guaranteed Delivery:** You must design for failure scenarios (e.g., What if a consumer misses an event?).




| Feature       | RabbitMQ                                | [Kafka](./Tools/Kafka.md#kafka)                           |
| ------------- | --------------------------------------- | ------------------------------- |
| Type          | Message broker (push)                   | Event log (pull-based)          |
| Typical usage | Real-time async processing, work queues | Event streaming, data pipelines |
| Persistence   | Optional durable queues                 | Persistent, append-only log     |
| Event replay  | Limited                                 | Built-in                        |                      |


# [RabbitMQ](https://www.cloudamqp.com/blog/part1-rabbitmq-for-beginners-what-is-rabbitmq.html)

![](https://www.cloudamqp.com/img/blog/exchanges-topic-fanout-direct.png)

<br />
<br />
<br />

---

# Leader election

Leader election is a fundamental concept in distributed systems where a group of nodes selects a single leader to coordinate tasks, ensuring order and consistency . Different algorithms and tools have been developed to solve this problem, each with its own strengths and trade-offs.

The table below summarizes some of the most prominent leader election algorithms and the tools that implement them.

| **Algorithm/Mechanism** | **How It Works** | **Key Tools & Systems That Use It** |
| :--- | :--- | :--- |
| **Bully Algorithm**  | The node with the highest unique ID wins. Nodes challenge others with higher IDs; if no response, they declare themselves leader. | |
| **Ring Algorithm**  | Nodes are logically arranged in a ring. An election message is passed; the node with the highest ID in the message becomes leader. | |
| **[Raft Consensus Algorithm](https://www.youtube.com/watch?v=IujMVjKvWP4&pp=ygUOcmFmdCBhbGdvcml0aG0%3D)**  | Nodes use randomized timeouts to become candidates and request votes. A node that receives a majority of votes becomes the leader. | **etcd** , **Consul**, **CockroachDB**  |
| **Paxos & Multi-Paxos**  | A node becomes leader by having its proposal accepted by a quorum (majority) of nodes. | |
| **ZooKeeper (ZAB Protocol)**  | Uses ephemeral sequential znodes. The node with the lowest sequence number is the leader. Watches notify the next node if the leader fails. | **Apache ZooKeeper** , **Kafka** (via ZooKeeper)  |
| **Lease-based Election**  | A leader acquires a lease (lock) from a shared database. It maintains leadership by periodically renewing the lease (heartbeating). | **Amazon DynamoDB** , **Apache ZooKeeper** , **Kinesis Client Library (KCL)**  |


# Some Scenerios in Raft

- New leader replicates its log to establish authority
- During replication, followers accept or reject entries based on log consistency
- The replication outcome naturally reveals which old entries were on a majority
- Committed entries are preserved; uncommitted ones are overwritten

## 1. Leader Commits an Entry and Crashes Before Followers Commit

### Scenario Breakdown
1.  A **Leader** receives a log entry from a client.
2.  The Leader appends the entry to its own log.
3.  The Leader sends **AppendEntries RPCs** to the **Followers**.
4.  The Leader receives successful responses from a **majority** of the Followers (e.g., 2 out of 3 servers).
5.  Based on the responses from the majority, the Leader **commits** the entry and applies it to its state machine (it may also send a success response to the client at this point).
6.  ***Crash Point:*** Before the Leader can send the next AppendEntries RPC, which would inform the remaining **minority** of Followers (or any lagging Followers) of the new commit index, the Leader **crashes**.

### Raft's Safety Mechanism

Raft guarantees that this committed entry will **not be lost** and will eventually be applied by *all* servers.

* **Electing a New Leader:** A new leader election is triggered. A new Leader will only be elected if it is **up-to-date** enough to contain **all committed entries**. Since the crashed Leader had the committed entry, and it had replicated it to a **majority** of servers, at least **one** of the servers in that majority must participate in the election (because any majority set overlaps with any other majority set, including the set of all servers).
* **Log Consistency:** The new Leader's $\text{AppendEntries}$ RPCs will force the minority/lagging Followers to **become consistent** with its log.
    * The new Leader sends the committed entry to the lagging Followers.
    * Once a lagging Follower receives an $\text{AppendEntries}$ RPC from the new Leader with a $\text{LeaderCommit}$ index greater than the Follower's current $\text{CommitIndex}$, the Follower commits and applies the entry.

**Conclusion:** The committed entry is **safe** because it existed on a majority of servers before the crash. The new Leader, being one of the majority, will use its log to **force consistency** across the cluster. 

---

## 2. Leader Fails an Entry Write, Tells Client Failure, and Crashes Before Telling Follower

### Scenario Breakdown
1.  A **Leader** receives a log entry from a client.
2.  The Leader appends the entry to its own log.
3.  The Leader sends $\text{AppendEntries}$ RPCs to the **Followers**.
4.  The Leader **fails** to get a response from a **majority** of Followers (e.g., Follower A and Follower B crash, but Follower C is up).
5.  The Leader **determines the write failed** and sends a **failure response** to the client. The entry is **uncommitted** and **unapplied**.
6.  ***Crash Point:*** Before the Leader can send the *next* $\text{AppendEntries}$ RPC or heartbeat to a specific follower (say Follower C, which had already received and stored the uncommitted entry), the Leader **crashes**.

### Raft's Safety Mechanism

The entry in question is **uncommitted** and must not be applied to the state machine. Raft ensures this uncommitted entry is either committed by the new leader or **deleted**.

* **Electing a New Leader:** A new Leader is elected.
* **Log Consistency and Rollback:** The new Leader may or may not have this uncommitted entry.
    * **Case A: The new Leader *does not* have the uncommitted entry.** The new Leader's log is **shorter** or **conflicts** with the log of the Follower (Follower C) that *does* have the uncommitted entry. The $\text{AppendEntries}$ RPC from the new Leader will use the **Log Matching Property** (matching $\text{term}$ and $\text{index}$) to find the point of divergence. The new Leader will force the Follower (Follower C) to **rollback** its log, deleting the uncommitted entry.
    * **Case B: The new Leader *does* have the uncommitted entry.** This can happen if the original Leader crashed, but was elected again (unlikely but possible), or if the new Leader's log is identical. The entry **remains uncommitted** on the new Leader. The new Leader will continue replication, and it will only be committed if it is replicated to a **majority** of servers *by the new Leader*.

**Key Point:** Since the Leader only failed to replicate the entry to a majority, the entry **never became committed**. Therefore, any new, legitimately elected Leader will either:
1.  **Discard** the entry if it's not present on the new Leader.
2.  **Maintain** the entry as **uncommitted** if it is present, and then attempt to replicate it anew. It will **only be committed** if it achieves a majority under the new Leader's term.

**Conclusion:** The entry remains **uncommitted** because it didn't meet the majority rule. The $\text{Log Matching Property}$ of $\text{AppendEntries}$ RPCs ensures that the new Leader's log will **always override** and **correct** the logs of all Followers, deleting any inconsistent, uncommitted entries.