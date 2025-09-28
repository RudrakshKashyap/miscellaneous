# **Monolithic vs. Microservices Architecture**

*   **Monolithic Architecture:** Builds an application as a single, unified unit with a single codebase. It offers simpler initial development and deployment but faces challenges with scalability and maintenance as it grows.
*   **Microservices Architecture:** Decomposes an application into small, independent services that communicate via well-defined APIs. It provides greater flexibility, independent scaling, and better fault isolation but introduces higher infrastructure complexity.

---

### **Monolithic Architecture**

| Aspect | Description |
| :--- | :--- |
| **Structure** | A single, large application where all components are tightly integrated and share a common codebase. |
| **Pros** | - **Simpler Development:** Easier to set up and manage during the early stages.<br>- **Straightforward Deployment:** Deployed as a single package or unit.<br>- **Centralized Management:** Easier to manage code and resources from a single point. |
| **Cons** | - **Scalability Issues:** Scaling any component requires scaling the entire application.<br>- **Slower Development:** Changes require rebuilding, retesting, and redeploying the entire monolith.<br>- **Technology Lock-in:** Restricted to a single technology stack.<br>- **Lower Resilience:** A single bug can potentially crash the entire application. |

---

### **Microservices Architecture**

| Aspect | Description |
| :--- | :--- |
| **Structure** | A collection of small, independent, and loosely coupled services. Each service handles a specific business function and often manages its own database. |
| **Pros** | - **Independent Scaling:** Each service can be scaled based on its specific load.<br>- **Increased Agility:** Teams can develop, deploy, and update services independently.<br>- **Better Fault Isolation:** A failure in one service is contained and doesn't bring down the whole system.<br>- **Technology Diversity:** Different services can use different technologies best suited for their tasks. |
| **Cons** | - **Increased Complexity:** Requires complex infrastructure for service communication, discovery, and monitoring.<br>- **Higher Initial Costs:** Significant investment needed for distributed systems infrastructure.<br>- **Data Consistency Challenges:** Maintaining transactions and data consistency across services is difficult.<br>- **Complex Testing:** Testing interactions between distributed services is more intricate. |

<br />
<br />
<br />


# [CAP Theorem](https://www.youtube.com/watch?v=VdrEq0cODu4)

The CAP theorem states that a distributed data store cannot simultaneously provide all three of the following guarantees: Consistency (all nodes see the same, most recent data), Availability (every request gets a response), and Partition Tolerance (the system continues operating despite network partitions). In a real-world distributed system experiencing a network failure, designers must make a trade-off, choosing to prioritize either Consistency (CP system) or Availability (AP system). 

![](https://media.licdn.com/dms/image/v2/D4D12AQFVB6hCSAcesQ/article-cover_image-shrink_720_1280/article-cover_image-shrink_720_1280/0/1712253012129?e=2147483647&v=beta&t=jH-I4wKbnirfPRfu4DcxkaDg-SK9bT-EigYWKyrEfUA)

## A Non Partition Tolerable system

**Scenario: A Bank Transfer**
*   Account X has $100.
*   Account Y has $100.

1.  **The Partition Occurs:** The network link between Node A and Node B fails. Neither node is aware of this. Both think the other is just slow to respond.

2.  **"Erroneously Continue":**
    *   A client connects to **Node A** and transfers **$50 from X to Y**.
        *   Node A updates its local state: `X = $50`, `Y = $150`.
        *   It tries to send this update to Node B, but the network is broken, so it fails. Node A doesn't care; it assumes the update will get there eventually and tells the client "Success!"
    *   *At the exact same time*, another client connects to **Node B** (which is still accepting writes because it's naive) and transfers **$20 from Y to X**.
        *   Node B's state is still `X = $100`, `Y = $100`. It applies the transfer: `X = $120`, `Y = $80`.
        *   It also tries to send this update to Node A and fails.

3.  **The State is Now Permanently Divergent:**
    *   **Node A believes:** `X = $50`, `Y = $150`
    *   **Node B believes:** `X = $120`, `Y = $80`
    *   The total money in the system is now different on each node ($200 on A, $200 on B). More importantly, the accounts are completely wrong.

4.  **The Partition is Repaired:** The network cable is plugged back in. Now Node A and Node B can talk again. They sync up.

**What does "normal" look like now? There is no "normal"!**

The system has two completely conflicting versions of history. It has to answer an impossible question:
*   Which transaction happened first, the $50 transfer or the $20 transfer?
*   Should the final state be based on Node A's timeline or Node B's?
*   How does it merge these two conflicting realities?

This is known as the **"split-brain"** scenario. The system has no automatic way to resolve this. The result is:
*   **Data Loss:** One of the two transactions will be silently overwritten and lost. Did $50 move? Or did $20 move? The bank's books will be wrong.
*   **Data Corruption:** The final state will be nonsensical. Maybe `X` will have $50 and `Y` will have $80, which means $70 simply vanished from the system with no record.
*   **No Consistency Guarantee:** The system is neither Consistent nor Available in the CAP sense, as it provided erroneous answers during the partition.

### Contrast with a Partition-Tolerant (CP or AP) System

A well-designed system *anticipates* partitions and has rules to prevent this corruption:

*   **A CP System:** Would stop accepting writes. The moment the partition was detected, one side (or both) would say, "I no longer have a quorum of nodes to talk to. I cannot guarantee my decisions are correct. I will become unavailable (return an error) until the network is healed." This prevents corruption at the cost of downtime.
*   **An AP System:** Would have rules for conflict resolution. For example, it might "last write wins," or allow both transactions to exist as conflicting versions and flag them for a human to resolve later. The key is that this inconsistency is a **known, managed state**, not a silent corruption.

**In summary: The problem isn't the temporary outage. The problem is the permanent data corruption that the temporary outage causes in a naive system.** "Fixing the network" doesn't undo the conflicting decisions that were made; it just reveals the mess that now has to be cleaned up, often manually. Partition-tolerant systems are designed specifically to avoid creating this mess in the first place.

<br />
<br />
<br />

# [Stateful vs Stateless Architectures](https://youtu.be/20tpk8A_xa0?si=0D1FBK4rXOnIrPZ5)

Stateless saves data in database so that any of the available nodes/servers can serve the request while in Statefull system, server stores the data(eg. SSH) and if it goes down, your data is lost.

The modern standard, especially for APIs and microservices, is overwhelmingly **stateless**. Technologies like **JSON Web Tokens (JWT)** allow you to get the best of both worlds:

1.  **Stateless Server:** The server does not store a session. It simply validates the signature of the JWT token provided in the request header.
2.  **Encoded State:** The token itself (usually stored on the client) contains encoded information (claims) about the user (e.g., user ID, roles). This provides the necessary "state" for the request without the server having to remember anything.

This approach combines the scalability and reliability of stateless architectures with the convenience of not having to send full credentials with every request.
---

<br />
<br />
<br />



# [REpresentational State Transfer (REST)]()
