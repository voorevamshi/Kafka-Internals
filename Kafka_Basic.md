## Kafka Broker Internal Architecture  Each broker contains:
A Kafka broker is a server in a Kafka cluster responsible for storing data, handling
client requests, and replicating partitions across brokers.
Each broker contains several internal components responsible for networking, storage, replication, and coordination.

```Code
Kafka Broker
 ├── Topic Partitions (stored on disk)
 ├── Replica Manager
 ├── Log Manager
 ├── Network Layer
 ├── Controller (if elected)
 ├── Group Coordinator
 └── Request Handler Threads
 

```
# 1. Log (Storage Layer)

Each **partition** in Kafka is stored as an **append-only log file** on disk.
Example directory structure:
```
/kafka-logs/
   topicA-0/
   topicA-1/
```
Each folder represents one **partition**.
### Files stored inside a partition

| File         | Purpose                                 |
| ------------ | --------------------------------------- |
| `.log`       | Stores actual messages                  |
| `.index`     | Maps offsets to message position in log |
| `.timeindex` | Maps timestamps to offsets              |

Example:

```
topicA-0/
   00000000000000000000.log
   00000000000000000000.index
   00000000000000000000.timeindex
```

Kafka writes data **sequentially** to these logs which makes it extremely fast.

---

# 2. Replica Manager

The **Replica Manager** manages **partition replication and consistency across brokers**.
### Responsibilities
#### Replication Between Brokers
Ensures follower replicas copy data from the leader partition.


Replication Factor of a topic means how many copies of each partition are stored across brokers in the Kafka cluster.

If a topic has:Partitions: 3 and Replication Factor: 2
Then Kafka stores 2 copies of each partition.
So total replicas = 3 partitions × 2 replicas = 6

| Partition | Leader Broker | Follower Broker |
| --------- | ------------- | --------------- |
| P0        | Broker1       | Broker2         |
| P1        | Broker2       | Broker3         |
| P2        | Broker3       | Broker1         |

Recommended Replication Factor  Production is 3, stage is 2 test is 1.
Kafka, the default replication factor for a topic is 1



Followers continuously fetch new messages from the leader.

#### Manage ISR (In-Sync Replicas)

ISR = replicas fully caught up with the leader.

Example:

| Replica | Status      |
| ------- | ----------- |
| Broker1 | Leader      |
| Broker2 | In Sync     |
| Broker3 | Out of Sync |

The Replica Manager updates the ISR list and removes slow replicas.

#### Leader–Follower Synchronization

Ensures followers stay close to the leader's offset.

#### Handle Leader Failures

If the leader fails:

1. Controller selects a new leader from ISR
2. Replica Manager activates the new leader

Flow:

```
Producer → Leader Partition → Followers replicate
```


---

# 3. Log Manager

The **Log Manager** manages how Kafka stores and maintains partition logs on disk.

### Responsibilities

#### Managing Log Segments

Kafka divides partition logs into **multiple segment files**.

Example:

```
orders-0/
   00000000000000000000.log
   00000000000000010000.log
   00000000000000020000.log
```

Each segment contains a portion of the log.

Benefits:

* Faster reads
* Easier deletion
* Efficient storage management

#### Log Retention

Old data is deleted based on retention policies.

Common configurations:

| Config          | Description                               |
| --------------- | ----------------------------------------- |
| retention.ms    | Delete messages older than specified time |
| retention.bytes | Delete when log exceeds configured size   |

Example:

```
retention.ms=604800000
```

Messages older than **7 days** are removed.

#### Log Compaction

When a topic uses:

```
cleanup.policy=compact
```

Kafka keeps **only the latest value for each key**.

Example:

| Key   | Value |
| ----- | ----- |
| user1 | A     |
| user1 | B     |

After compaction:

```
user1 = B
```

#### Disk Storage Management

Log Manager handles:

* Writing messages to disk
* Rolling new log segments
* Cleaning old segments
* Managing multiple log directories

Example configuration:

```
log.dirs=/data1/kafka-logs,/data2/kafka-logs
```

Kafka distributes partitions across available disks.

---

# 4. Network Layer

Handles all communication between:

* Producers
* Consumers
* Other brokers

The network layer accepts requests over TCP and forwards them to request handler threads.

---

# 5. Controller (If Broker Is Elected)

Only **one broker acts as controller** in a Kafka cluster.

Responsibilities:

* Leader election
* Broker failure detection
* Partition reassignment
* Topic creation and deletion coordination

If the controller broker fails, another broker is elected automatically.

---

# 6. Group Coordinator

The **Group Coordinator** manages consumer groups.

Responsibilities:

* Consumer membership tracking
* Partition assignment to consumers
* Managing consumer heartbeats
* Handling offset commits

Consumer offsets are stored in the internal topic:

```
__consumer_offsets
```

---

# 7. Request Handler Threads

Request handler threads process incoming client requests.

Examples of requests:

* Produce requests
* Fetch requests
* Metadata requests
* Offset commit requests

Flow:

```
Client Request → Network Layer → Request Handler → Kafka Component
```

---

# Kafka Message Flow (Simplified)

```
Producer
   ↓
Network Layer
   ↓
Request Handler Threads
   ↓
Replica Manager
   ↓
Log Manager
   ↓
Partition Log (Disk)
   ↓
Followers Replicate
```

---

# Quick Interview Summary

| Component         | Role                                         |
| ----------------- | -------------------------------------------- |
| Log               | Stores partition messages on disk            |
| Replica Manager   | Handles replication and ISR                  |
| Log Manager       | Manages segments, retention, compaction      |
| Network Layer     | Handles client connections                   |
| Controller        | Manages cluster metadata and leader election |
| Group Coordinator | Manages consumer groups                      |
| Request Handlers  | Processes client requests                    |

---

# Key Interview One-Liner

**A Kafka broker stores partition logs and internally uses components like Replica Manager, Log Manager, Network Layer, Controller, and Group Coordinator to handle replication, storage, and client communication.**


