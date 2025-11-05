
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
