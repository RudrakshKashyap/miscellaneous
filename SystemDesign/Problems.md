# Rate Limiter
- https://www.youtube.com/watch?v=YXkOdWBwqaA
- https://www.youtube.com/watch?v=MIJFyUPG4Z4


# Distributed Lock

**Lock should have atomic "check and set" as single operation.**

The core issue is that Redis's locking algorithms, including Redlock, **do not natively generate fencing tokens** .


- **For Banking and Other Correctness-Critical Applications**: **Do not use Redis distributed locks.(even with fencing tokens)** The risks of data corruption(bc of redis async nature), however small, are fundamentally unacceptable. You should instead use a system designed from the ground up for strong consistency and reliable coordination, such as **ZooKeeper** or **etcd** . These systems use consensus algorithms and provide the necessary primitives, like fencing tokens, out-of-the-box.

[![](https://martin.kleppmann.com/2016/02/fencing-tokens.png)](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)

- **The "Zombie Process" Problem**: In a distributed system, a process can be paused for many reasons (e.g., garbage collection, network delays). If a process holding a lock is paused longer than the lock's time-to-live (TTL), the lock will expire and be given to another process. When the first process resumes, it may unknowingly no longer hold the lock and proceed to perform an unsafe action, corrupting data .
- **The Solution**: A **fencing token** is a monotonically increasing number issued by the lock service with every successful lock acquisition. The resource (e.g., your database) must check this token, rejecting any write with a token less than the last accepted one. This ensures that a "zombie" process with a stale token cannot perform harmful operations .
