## Each broker contains:
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
🔹 1. Log (Storage Layer)

### Each partition = append-only log file.
```code
/kafka-logs/
   topicA-0/
   topicA-1/
```

### Kafka stores:

- Messages

- Offsets

- Index files

### 🔹 2. Replica Manager  Handles:

- Leader/Follower replication

- ISR (In-Sync Replicas)

- Replication between brokers


### 🔹 3. Network Layer Handles:

- Producer requests

- Consumer fetch requests

- Metadata requests

- Uses TCP-based protocol.

### 🔹 4. Controller (One per Cluster)

- One broker becomes Controller.

#### Controller handles:

- Leader election

- Partition reassignment

- Broker failure detection

### 🔹 5. Group Coordinator Manages:

- Consumer groups

- Rebalancing

- Offset tracking


### 2️⃣ Role of Kafka Broker vs Bootstrap Server
### 🔵 Kafka Broker Role

- Stores data (topics & partitions)

- Handles read/write

- Manages replication

- Ensures durability

- Serves metadata

- It is the actual data server.

### 🟢 Bootstrap Server Role

- It only helps client discover cluster metadata.

#### Flow:
```code
Producer → Connect to bootstrap server
Bootstrap → Returns metadata (all brokers, leaders)
Producer → Connects to actual leader broker
```


### End-to-End Flow (Visual)

```code
Producer
   ↓
Bootstrap Broker
   ↓ (metadata)
Leader Broker (Partition 0)
   ↓
Followers replicate
   ↓
Consumer fetches from Leader

```

