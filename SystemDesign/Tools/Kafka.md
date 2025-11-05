# [Kafka](https://www.hellointerview.com/learn/system-design/deep-dives/kafka#fault-tolerance-and-durability)

Kafka is a distributed event streaming platform designed to handle high volumes of real-time data with high throughput, fault tolerance, and scalability .

[![](https://images.ctfassets.net/gt6dp23g0g38/4DA2zHan28tYNAV2c9Vd98/b9fca38c23e2b2d16a4c4de04ea6dd3f/Kafka_Internals_004.png)](https://developer.confluent.io/courses/architecture/get-started/)

### Core Architectural Components

The foundation of Kafka's architecture is built on several key components that work together to store and move streams of data reliably.

| **Component** | **Description** |
| :--- | :--- |
| **Topics & Partitions** | A **Topic** is a categorized feed to which records are published . Each topic is split into one or more **Partitions/Queues**, which are immutable, ordered sequences of records . |
| **Producers** | Applications that publish (write) records to Kafka topics . |
| **Consumers & Consumer Groups** | Applications that subscribe to (read) records from topics . **Consumer Groups** allow a pool of processes to divide the work of consuming records . |
| **Brokers & Clusters** | A **Broker** is a single Kafka server that stores data and serves clients. A **Cluster** is a group of one or more brokers working together . |
| **ZooKeeper & KRaft** | **ZooKeeper** was historically used for managing cluster metadata and coordinating brokers . Newer versions of Kafka are replacing this with **KRaft**, a built-in consensus mechanism, for simpler operations and better scalability . |

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*kM_cRlgbrRKG5-zT97qF1Q.jpeg)

### The Anatomy of a Kafka Record

When an application writes data to Kafka, it creates a **Record** (also called a message or event) . A record consists of several parts:
- **Key**: An optional identifier, often used to assign the record to a specific topic partition .
- **Value**: The actual payload or data of the record .
- **Timestamp**: The time when the event occurred .
- **Headers**: Optional key-value pairs for storing additional metadata .

Kafka uses a **schema** (e.g., Avro, Protobuf) to structure the key and value, often managed by a **Schema Registry** to ensure compatibility between producers and consumers .

### How Data Flows in Kafka

#### 1. Data Production and Partitioning
Producers publish records to topics. The key decision is determining which partition the record goes to :
- If a **key is provided**, a hash of the key determines the partition, guaranteeing that all records with the same key go to the same partition, thus preserving order for that key .
- If **no key is provided**, records are distributed across partitions in a round-robin fashion for load balancing .

Producers can be configured for different **acknowledgment (acks)** levels to balance durability and latency :
- **`acks=0`**: The producer does not wait for a response (possible data loss).
- **`acks=1`**: The producer waits for the partition leader to acknowledge (limited data loss).
- **`acks=all`**: The producer waits for the leader and all in-sync replicas to acknowledge (no data loss).

#### 2. Data Storage and Replication
- **Append-Only Log**: Each partition is an ordered, append-only log stored on disk. Records are assigned a unique, sequential ID called an **offset** .
- **Replication for Fault Tolerance**: Each partition is replicated across multiple brokers. One broker acts as the **leader**, handling all reads and writes for that partition, while the others are **followers** that replicate the data . If the leader fails, a follower is promoted to a new leader automatically, ensuring high availability. Kafka’s replication mechanism **EXPLICITLY PREVENTS** multiple replicas of the same partition from being placed on the same broker. Otherwise replication wouldn’t offer any fault tolerance. If the broker failed, both the leader and its follower would be lost together, defeating the purpose.

#### 3. Data Consumption
- **Pull Model**: Consumers actively poll brokers for new records, allowing them to control their consumption rate .
- **Consumer Groups**: To scale consumption, consumers are organized into groups. Each partition is consumed by **only one consumer** within the group, enabling parallel processing. You cannot have more consumers in a group than partitions, or the extra consumers will be idle .
- **Offset Tracking**: Consumers track their position in the partition log using the offset. They periodically **commit** their progress so they can resume from that point after a restart .

### Kafka's APIs

Kafka provides five core APIs for different functions :
1.  **Admin API**: To manage and inspect topics, brokers, and other Kafka objects.
2.  **Producer API**: To publish (write) a stream of records to one or more Kafka topics.
3.  **Consumer API**: To subscribe to (read) one or more topics and process their streams of records.
4.  **Kafka Streams API**: A Java library for building real-time streaming applications that can perform complex processing, transformations, and aggregations on data from Kafka .
5.  **Kafka Connect API**: A framework for building and managing reusable connectors that efficiently integrate Kafka with external systems like databases (e.g., using source connectors to ingest data and sink connectors to export data) .

### Key Mechanisms for Performance and Reliability

- **Message Ordering**: Kafka guarantees ordered delivery of records **only within a single partition** . Order across the entire topic is not guaranteed.
- **Message Retention**: Records in Kafka are persisted to disk and kept for a configurable amount of time (default is one week), not deleted after consumption. This allows multiple consumers to read the same data and to replay events if needed.
- **Delivery Semantics**: Kafka supports different levels of delivery guarantees :
    - **At-least-once**: Records are never lost but may be redelivered (duplicates possible).
    - **At-most-once**: Records may be lost but are never redelivered.
    - **Exactly-once**: Achieved through idempotent producers and transactional writes, ensuring each record is processed once and only once.


# TODO - https://chatgpt.com/c/68eb9c70-47a8-8322-aee0-1429f26a716f
