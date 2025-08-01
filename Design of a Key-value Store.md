# Design of a Key-value Store

## Overview
A key-value store is a type of NoSQL database that stores data as a collection of key-value pairs, where a key serves as a unique identifier and is associated with a value.

## Functional Requirements
- **put(key, value)**: Store key-value pair
- **get(key)**: Retrieve value associated with key
- **delete(key)**: Remove key-value pair

## Non-functional Requirements
- **Scalability**: Handle large amounts of data and high traffic
- **Availability**: System should be highly available
- **Consistency**: Data consistency across replicas
- **Performance**: Low latency for reads and writes
- **Durability**: Data should not be lost

## API Design

### Basic Operations
```
PUT /v1/data/{key}
Body: {value}
```

```
GET /v1/data/{key}
Response: {value}
```

```
DELETE /v1/data/{key}
```

## High-Level Design

### Single Server
- **Hash table in memory**
- **Limitations**: Memory constraints, no fault tolerance
- **Use case**: Small datasets, development/testing

### Distributed System Components

#### Load Balancer
- **Distributes client requests**
- **Health checking of servers**
- **SSL termination**

#### Key-Value Servers
- **Store actual data**
- **Handle read/write operations**
- **Partitioning logic**

#### Metadata Service
- **Track server health**
- **Manage configuration**
- **Handle cluster membership**

## Data Partitioning

### Consistent Hashing
- **Maps keys to hash ring**
- **Servers placed on ring**
- **Key stored on first server clockwise**
- **Benefits**: Minimal data movement when adding/removing servers

### Virtual Nodes
- **Each physical server gets multiple positions on ring**
- **Better load distribution**
- **Easier rebalancing**

## Replication Strategy

### Replication Factor
- **N replicas for each key**
- **First replica on designated server**
- **Next N-1 replicas on clockwise servers**

### Consistency Models
- **Strong Consistency**: All replicas updated before response
- **Eventual Consistency**: Replicas updated asynchronously
- **Weak Consistency**: No guarantees on when replicas sync

## Consistency Protocols

### Quorum-based
- **N**: Number of replicas
- **W**: Write quorum (successful writes needed)
- **R**: Read quorum (replicas to read from)
- **Strong consistency**: R + W > N**
- **Fast reads**: R = 1, W = N
- **Fast writes**: W = 1, R = N

### Vector Clocks
- **Track causality between events**
- **Detect concurrent updates**
- **Resolve conflicts**

## Failure Handling

### Detection
- **Heartbeat mechanism**
- **Gossip protocol for membership**
- **Health checks from load balancer**

### Resolution
- **Temporary failures**: Hinted handoff
- **Permanent failures**: Anti-entropy with Merkle trees
- **Network partitions**: Sloppy quorum

## System Architecture

### Storage Engine
- **LSM Trees**: Good for write-heavy workloads
- **B+ Trees**: Good for read-heavy workloads
- **Write-Ahead Log (WAL)**: Durability guarantee

### Caching Layer
- **In-memory cache for hot data**
- **LRU eviction policy**
- **Cache-aside pattern**

### Monitoring
- **Metrics collection**
- **Health dashboards**
- **Alerting system**

## Trade-offs

### CAP Theorem
- **Consistency vs Availability** during network partitions
- **Most key-value stores choose AP** (Availability + Partition tolerance)

### Performance vs Consistency
- **Async replication**: Better performance, weaker consistency
- **Sync replication**: Stronger consistency, higher latency

### Storage vs Speed
- **In-memory**: Fast but expensive and volatile
- **SSD**: Good balance of speed and cost
- **HDD**: Cheap but slower

## Real-world Examples
- **Amazon DynamoDB**: Managed, multi-model
- **Apache Cassandra**: Wide-column, high availability
- **Redis**: In-memory, various data structures
- **Riak**: Distributed, fault-tolerant