# Data-Intensive Applications (DDIA)
This document describes the essence of Data-Intensive Applications and the recommendations and deprecations that support it.

## Essence of Data-Intensive Applications

In one sentence, Data-Intensive Applications is about building systems where data problems—the quantity, complexity, and speed of change—are the primary challenges rather than compute power. When building such systems, follow these philosophies:

- [ ] Pursue reliability, scalability, and maintainability as the fundamental goals of system design
- [ ] Accept the reality of distributed systems—partial failures and non-determinism are inevitable
- [ ] Understand the essence of data consistency and ordering, choosing appropriate guarantee levels based on requirements

## Recommendations

**Build systems considering reliability, scalability, and maintainability**:

These are the actual goals to achieve in designing data-oriented applications, forming the foundation for system health, sustainability, and evolution. In modern applications where data volume, complexity, and rate of change are major challenges, these qualities determine business continuity, user experience quality, and development team productivity.

Focus on three key aspects:
- Reliability: Systems must continue working correctly despite hardware failures, software errors, and human mistakes
- Scalability: Systems must handle growth in data volume, traffic, and complexity through reasonable approaches
- Maintainability: Systems must allow many people to work productively over time through good operability, simplicity, and evolvability

**Design for fault tolerance in systems**:

Systems can experience faults that may lead to failures. Since it's impossible to reduce fault probability to zero, design fault-tolerant mechanisms. Consider hardware faults (disk failures, RAM errors, power outages), software errors (bugs, runaway processes, cascading failures), and human errors. Implement systematic error handling, thorough testing, process isolation, detailed monitoring, and quick recovery mechanisms. Tools like Netflix's Chaos Monkey can help test fault tolerance by intentionally causing failures.

**Manage system complexity and pursue simplicity**:

As projects grow, code becomes complex and harder to understand. This complexity slows down everyone involved and increases maintenance costs. Symptoms include state space explosion, tight coupling between modules, tangled dependencies, inconsistent naming, performance hacks, and special-case handling. Combat this through good abstractions that hide implementation details, remove accidental complexity (complexity not inherent to the problem being solved), and enable reusable components.

**Understand storage engine internals**:

Application developers need to understand how storage engines work to select appropriate ones and tune performance for their workloads. Key distinctions include:
- Transaction processing vs analytics workloads require different optimizations
- LSM-trees typically handle writes faster while B-trees excel at reads
- Benchmarks depend heavily on workload details—test with your specific workload

**Consider replication lag impact in eventually consistent systems**:

Replication lag can grow to minutes or hours, making inconsistency a real problem rather than theoretical. Issues include users not seeing data they just wrote, time-traveling queries, and causality violations. While applications can provide stronger guarantees than the underlying database, this adds complexity. Understand techniques like read-after-write consistency, monotonic reads, and consistent prefix reads to handle these challenges appropriately.

**Assume worst-case scenarios in distributed systems**:

"Anything that can go wrong will go wrong" is a realistic assumption. Distributed systems introduce numerous failure modes: packet loss, reordering, duplication, network delays, clock inaccuracies, node pauses, and crashes. Engineers must build systems that work correctly despite everything going wrong. Use coordination services like Apache ZooKeeper or etcd for distributed locks and leader election. Implement fencing tokens to prevent confused nodes from corrupting the system.

**Understand and apply atomic transaction principles**:

Transactions group multiple reads and writes into logical units, simplifying error handling and concurrency issues. Understand ACID properties (Atomicity, Consistency, Isolation, Durability) and their implementations, which vary between databases. Key concepts include:
- Atomicity ensures all-or-nothing execution with safe retry capability
- Isolation levels (read committed, snapshot isolation, serializability) prevent different race conditions
- Serializability provides the strongest guarantees but has performance costs
- Serializable Snapshot Isolation (SSI) offers full serializability with better performance than traditional approaches

**Apply dataflow and immutability principles**:

Many systems generate data continuously over time, making stream processing essential. Input immutability improves batch and stream processing robustness and maintainability. Key practices include:
- Treat database changes as event streams through change data capture (CDC) or event sourcing
- Use log compaction to manage change history efficiently
- Distinguish commands (requests) from events (facts that happened)
- Achieve exactly-once semantics through idempotent operations and proper failure handling
- Store events permanently when possible, enabling complete reprocessing when needed

**Prioritize consistency over timeliness**:

Timeliness violations are temporary (eventual consistency) and resolve with waiting, but consistency violations lead to permanent data corruption, loss, or contradictions. In most applications, consistency is far more important. While ACID transactions typically guarantee both, event-based dataflow systems can separate these concerns. Design systems following the end-to-end principle, where applications handle guarantees like deduplication themselves for true fault tolerance.

**Combine multiple data systems for consistent application architecture**:

Large applications cannot satisfy all data access, query, and processing requirements with a single database. Use appropriate tools for each job:
- Leverage batch processing for search indexes, recommendations, and analytics
- Use message brokers for reliable asynchronous communication
- Implement log-based message brokers combining database persistence with low-latency notifications
- Apply event sourcing to capture system state changes as immutable event logs
- Use stream processing for monitoring, real-time dashboards, and continuous data synchronization

## Deprecations

**Avoid treating CAP theorem as practically useful**:

CAP theorem has narrow scope and creates confusion. It's concluded to have little practical value in system design. Instead, evaluate linearizability, availability, and network partitions as individual tradeoffs with specific implications for your system.

**Avoid simply hoping for the best**:

Distributed systems can experience wide-ranging faults, including highly improbable ones. Accept the possibility of partial failures and build fault tolerance mechanisms into software. Intentionally cause network failures in test environments to verify system responses.

**Avoid synchronous replication to all followers**:

One node failure or network disruption would make the entire system unable to accept writes. Reliability decreases dramatically as node count increases. Instead, use asynchronous replication for read scaling and carefully design application behavior to handle replication lag.

**Avoid pretending asynchronous replication is synchronous**:

This deception will eventually cause problems affecting data consistency and durability. Design systems to provide stronger guarantees like read-after-write consistency while acknowledging asynchronous nature in application architecture.

**Avoid weak isolation levels carelessly**:

Weak isolation levels don't prevent all concurrency problems. Timing-dependent bugs like dirty reads, lost updates, and write skew are hard to detect and have caused significant financial losses and data corruption. Deeply understand concurrency issues and consider serializability for applications requiring reliability and correctness.

**Avoid language-specific serialization formats**:

These formats create problems: difficulty integrating with other languages, security vulnerabilities (arbitrary class instantiation), poor versioning support, and ignored compatibility concerns. Use schema-based binary encoding formats like Thrift, Protocol Buffers, or Avro instead.

**Avoid trusting user-controlled device clocks**:

Users may set incorrect times intentionally, and virtual machine environments can cause clock pauses. For event timestamps, record three times: event occurrence (device clock), server send time (device clock), and server receive time (server clock). Estimate accurate occurrence times by calculating device clock offsets server-side.

**Avoid hash modulo partitioning**:

Using hash(key) mod N causes massive data movement when node count changes. Most keys must move to different partitions, making rebalancing extremely expensive. Instead, divide hash ranges into partitions and assign multiple partitions per node for efficient dynamic rebalancing.

**Avoid long-running transactions**:

In serializable isolation implementations like two-phase locking (2PL), long transactions block others for extended periods, destabilize latency, and increase deadlock frequency. Keep transactions short. Use stored procedures to complete multiple operations quickly within transactions rather than waiting for user input.

**Avoid making applications handle all error recovery in leaderless datastores**:

Leaderless stores operate "best effort" and don't roll back on errors, making error handling extremely complex. Use transactions providing atomicity (rollback on errors) and isolation (preventing concurrent transaction interference) to simplify error handling.

**Avoid XA transactions as universal solutions**:

XA has significant operational problems: coordinators become single points of failure, application servers lose statelessness, features are limited to lowest common denominator, and failures tend to amplify across systems. Consider log-based derived data and stream processing approaches for consistency across heterogeneous systems without distributed transaction pain.

**Avoid dual writes to multiple storage systems**:

When applications write directly to multiple systems (e.g., search index and database), concurrent conflicting writes processed in different orders cause permanent inconsistency. Route user input through a single system determining all write ordering, applying state machine replication principles for easier data derivation.

**Avoid reckless user data collection and usage**:

Personal data is valuable but carries risks of abuse, unauthorized access, and breaches. Data should be considered "toxic assets" or "hazardous material." Comply with data protection regulations, treat users as humans deserving respect rather than metrics to optimize, self-regulate data practices, educate users about data usage, enable privacy control, and delete unnecessary data promptly.