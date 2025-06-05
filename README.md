# Cassandra Interview Guide & Knowledge Base

A comprehensive guide covering Apache Cassandra fundamentals, architecture, operations, and interview preparation.

## Table of Contents

### 1. [Introduction](#introduction)
### 2. [Cassandra Fundamentals](#cassandra-fundamentals)
- [What is Cassandra?](#what-is-cassandra)
- [Key Features](#key-features)
- [Use Cases](#use-cases)

### 3. [Architecture Deep Dive](#architecture-deep-dive)
- [Distributed Architecture](#distributed-architecture)
- [Ring Topology](#ring-topology)
- [Consistent Hashing](#consistent-hashing)
- [Replication](#replication)
- [Gossip Protocol](#gossip-protocol)

### 4. [Data Model](#data-model)
- [Column Family/Table Structure](#column-familytable-structure)
- [Primary Key Components](#primary-key-components)
- [Partition Key vs Clustering Key](#partition-key-vs-clustering-key)
- [Data Types](#data-types)

### 5. [Consistency and CAP Theorem](#consistency-and-cap-theorem)
- [CAP Theorem in Cassandra](#cap-theorem-in-cassandra)
- [Consistency Levels](#consistency-levels)
- [Tunable Consistency](#tunable-consistency)

### 6. [Read and Write Operations](#read-and-write-operations)
- [Write Path](#write-path)
- [Read Path](#read-path)
- [Compaction](#compaction)
- [Bloom Filters](#bloom-filters)

### 7. [Performance and Optimization](#performance-and-optimization)
- [Performance Tuning](#performance-tuning)
- [Monitoring](#monitoring)
- [Best Practices](#best-practices)

### 8. [Operations and Maintenance](#operations-and-maintenance)
- [Cluster Management](#cluster-management)
- [Backup and Recovery](#backup-and-recovery)
- [Troubleshooting](#troubleshooting)

### 9. [CQL (Cassandra Query Language)](#cql-cassandra-query-language)
- [Basic Operations](#basic-operations)
- [Advanced Queries](#advanced-queries)
- [Limitations](#limitations)

### 10. [Interview Questions](#interview-questions)
- [Fundamental Questions](#fundamental-questions)
- [Architecture Questions](#architecture-questions)
- [Performance Questions](#performance-questions)
- [Scenario-based Questions](#scenario-based-questions)

---

## Introduction

Apache Cassandra is a highly scalable, distributed NoSQL database designed to handle large amounts of data across multiple commodity servers while providing high availability with no single point of failure.

## Cassandra Fundamentals

### What is Cassandra?

Apache Cassandra is a free, open-source, distributed, wide-column store NoSQL database management system designed to handle large amounts of data across many commodity servers, providing high availability with no single point of failure.

**Key Characteristics:**
- Distributed and decentralized
- Masterless architecture
- Linear scalability
- Fault tolerance
- Eventually consistent
- Column-oriented storage

### Key Features

- **High Availability**: No single point of failure
- **Linear Scalability**: Performance increases proportionally with nodes
- **Fault Tolerance**: Automatic data replication across multiple nodes
- **Flexible Data Model**: Wide-column store with dynamic schema
- **Tunable Consistency**: Choose between consistency and availability
- **Multi-datacenter Support**: Geographic distribution capabilities

### Use Cases

**Ideal for:**
- Time-series data (IoT sensors, logs)
- Real-time analytics
- Content management
- Social media applications
- Recommendation engines
- Fraud detection systems

**Not suitable for:**
- ACID transactions
- Complex joins
- Ad-hoc queries
- Small datasets
- Applications requiring strong consistency

## Architecture Deep Dive

### Distributed Architecture

Cassandra uses a **peer-to-peer architecture** where all nodes are equal. There's no master-slave relationship, eliminating single points of failure.

**Key Components:**
- **Nodes**: Individual servers in the cluster
- **Data Centers**: Logical grouping of nodes
- **Clusters**: Collection of data centers
- **Racks**: Physical grouping within data centers

### Ring Topology

Cassandra organizes nodes in a ring structure where:
- Each node is assigned a token range
- Data is distributed based on partition key hash
- Nodes communicate using gossip protocol
- Token ranges determine data ownership

### Consistent Hashing

Uses consistent hashing to:
- Distribute data evenly across nodes
- Minimize data movement during scaling
- Determine which nodes store specific data
- Handle node additions/removals efficiently

**Hash Function**: Murmur3 (default) produces 128-bit hash values

### Replication

**Replication Strategy:**
- **SimpleStrategy**: Single data center deployment
- **NetworkTopologyStrategy**: Multi-data center deployment

**Replication Factor (RF):**
- Defines number of copies of each piece of data
- Common values: RF=3 for production
- Affects consistency, availability, and storage requirements

### Gossip Protocol

**Purpose**: Node discovery and failure detection

**Process:**
1. Nodes exchange state information every second
2. Information propagates exponentially through the cluster
3. Includes node status, load, schema version
4. Enables automatic failure detection and recovery

## Data Model

### Column Family/Table Structure

```
Table (Column Family)
├── Row (identified by Partition Key)
│   ├── Partition Key
│   ├── Clustering Columns (optional)
│   └── Regular Columns
```

### Primary Key Components

**Primary Key = Partition Key + Clustering Key**

```sql
CREATE TABLE users (
    user_id UUID,           -- Partition Key
    timestamp TIMESTAMP,    -- Clustering Key
    name TEXT,             -- Regular Column
    email TEXT,            -- Regular Column
    PRIMARY KEY (user_id, timestamp)
);
```

### Partition Key vs Clustering Key

| Aspect | Partition Key | Clustering Key |
|--------|---------------|----------------|
| Purpose | Determines which node stores data | Orders data within partition |
| Distribution | Distributes data across nodes | Organizes data within partition |
| Queries | Required for efficient queries | Enables range queries |
| Uniqueness | Can have duplicates | Must be unique within partition |

### Data Types

**Basic Types:**
- `TEXT`, `VARCHAR`
- `INT`, `BIGINT`, `SMALLINT`, `TINYINT`
- `FLOAT`, `DOUBLE`, `DECIMAL`
- `BOOLEAN`
- `TIMESTAMP`, `DATE`, `TIME`
- `UUID`, `TIMEUUID`
- `BLOB`

**Collection Types:**
- `LIST<type>`
- `SET<type>`
- `MAP<key_type, value_type>`

**User-Defined Types (UDT)**

## Consistency and CAP Theorem

### CAP Theorem in Cassandra

Cassandra is **AP** (Available and Partition-tolerant) system:
- **Consistency**: Eventually consistent (tunable)
- **Availability**: Highly available
- **Partition Tolerance**: Handles network partitions gracefully

### Consistency Levels

**Write Consistency Levels:**
- `ONE`: Write to one replica
- `QUORUM`: Write to majority of replicas
- `ALL`: Write to all replicas
- `LOCAL_QUORUM`: Majority in local data center
- `EACH_QUORUM`: Majority in each data center

**Read Consistency Levels:**
- `ONE`: Read from one replica
- `QUORUM`: Read from majority of replicas
- `ALL`: Read from all replicas
- `LOCAL_QUORUM`: Majority in local data center

### Tunable Consistency

**Strong Consistency Formula:**
```
Read CL + Write CL > Replication Factor
```

**Examples:**
- RF=3, Write=QUORUM(2), Read=QUORUM(2) → Strong consistency
- RF=3, Write=ONE(1), Read=ONE(1) → Eventual consistency

## Read and Write Operations

### Write Path

1. **Client Request**: Write request received
2. **Coordinator Selection**: Any node can act as coordinator
3. **Partition Key Hashing**: Determine target nodes
4. **Replication**: Write to RF number of nodes
5. **Consistency Check**: Wait for required acknowledgments
6. **Response**: Return success/failure to client

**Write Process on Node:**
1. Write to commit log (durability)
2. Write to memtable (memory)
3. When memtable full, flush to SSTable (disk)

### Read Path

1. **Client Request**: Read request received
2. **Coordinator Selection**: Any node can coordinate
3. **Partition Key Hashing**: Determine target nodes
4. **Data Retrieval**: Read from required replicas
5. **Data Reconciliation**: Merge results using timestamps
6. **Response**: Return data to client

**Read Process on Node:**
1. Check bloom filters
2. Check key cache
3. Check row cache
4. Read from memtables
5. Read from SSTables (disk)
6. Merge and return results

### Compaction

**Purpose**: Merge SSTables and remove tombstones

**Strategies:**
- **SizeTieredCompactionStrategy (STCS)**: Default, good for write-heavy
- **LeveledCompactionStrategy (LCS)**: Good for read-heavy
- **TimeWindowCompactionStrategy (TWCS)**: Good for time-series data

### Bloom Filters

**Purpose**: Avoid unnecessary disk reads

**How it works:**
- Probabilistic data structure
- Tells if data is definitely NOT in SSTable
- Reduces false positives for missing data
- Configurable false positive probability

## Performance and Optimization

### Performance Tuning

**Hardware Recommendations:**
- **CPU**: 8+ cores, high clock speed
- **Memory**: 16GB+ RAM, avoid swap
- **Storage**: SSD preferred, separate disks for commit log
- **Network**: Gigabit Ethernet minimum

**JVM Tuning:**
- Use G1GC for heaps > 6GB
- Set Xms and Xmx to same value
- Monitor GC logs and tune accordingly

**Configuration Tuning:**
```yaml
# cassandra.yaml key settings
memtable_allocation_type: heap_buffers
memtable_heap_space_in_mb: 2048
memtable_offheap_space_in_mb: 2048
concurrent_reads: 32
concurrent_writes: 32
```

### Monitoring

**Key Metrics:**
- **Latency**: Read/write response times
- **Throughput**: Operations per second
- **Error Rate**: Failed operations percentage
- **Resource Utilization**: CPU, memory, disk I/O
- **Garbage Collection**: GC frequency and duration

**Tools:**
- DataStax OpsCenter
- Prometheus + Grafana
- nodetool commands
- JMX metrics

### Best Practices

**Data Modeling:**
- Design for your queries
- Avoid large partitions (>100MB)
- Use appropriate partition keys
- Minimize the number of partitions read
- Denormalize data when necessary

**Operations:**
- Regular backups
- Monitor cluster health
- Plan capacity carefully
- Use appropriate consistency levels
- Test disaster recovery procedures

## Operations and Maintenance

### Cluster Management

**Adding Nodes:**
```bash
# Set initial_token or use automatic token assignment
# Start new node
cassandra -f

# Check status
nodetool status
```

**Removing Nodes:**
```bash
# Decommission node
nodetool decommission

# Or force removal (if node is down)
nodetool removenode <node-id>
```

**Scaling Operations:**
- **Scale Up**: Add more nodes to increase capacity
- **Scale Out**: Increase replication factor for availability
- **Scale Down**: Remove nodes (carefully plan data migration)

### Backup and Recovery

**Snapshot Creation:**
```bash
# Create snapshot
nodetool snapshot keyspace_name

# List snapshots
nodetool listsnapshots

# Clear snapshot
nodetool clearsnapshot
```

**Backup Strategies:**
- **Full Backups**: Complete snapshot of all data
- **Incremental Backups**: Only new SSTables since last backup
- **Point-in-Time Recovery**: Combine snapshots with commit logs

### Troubleshooting

**Common Issues:**
- **Timeout Exceptions**: Increase timeout values, check network
- **Heap Memory Issues**: Tune JVM settings, check for memory leaks
- **High Latency**: Check disk I/O, compaction, garbage collection
- **Data Inconsistency**: Run repair operations, check consistency levels

**Diagnostic Commands:**
```bash
# Cluster status
nodetool status
nodetool info
nodetool ring

# Performance metrics
nodetool tpstats
nodetool cfstats
nodetool proxyhistograms

# Repair operations
nodetool repair
nodetool cleanup
```

## CQL (Cassandra Query Language)

### Basic Operations

**Keyspace Operations:**
```sql
-- Create keyspace
CREATE KEYSPACE mykeyspace 
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'datacenter1': 3
};

-- Use keyspace
USE mykeyspace;

-- Drop keyspace
DROP KEYSPACE mykeyspace;
```

**Table Operations:**
```sql
-- Create table
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT,
    created_at TIMESTAMP
);

-- Insert data
INSERT INTO users (user_id, name, email, created_at)
VALUES (uuid(), 'John Doe', 'john@example.com', toTimestamp(now()));

-- Select data
SELECT * FROM users WHERE user_id = ?;

-- Update data
UPDATE users SET email = 'newemail@example.com' 
WHERE user_id = ?;

-- Delete data
DELETE FROM users WHERE user_id = ?;
```

### Advanced Queries

**Secondary Indexes:**
```sql
-- Create index
CREATE INDEX ON users (name);

-- Query using index
SELECT * FROM users WHERE name = 'John Doe';
```

**Materialized Views:**
```sql
-- Create materialized view
CREATE MATERIALIZED VIEW users_by_email AS
SELECT * FROM users
WHERE email IS NOT NULL AND user_id IS NOT NULL
PRIMARY KEY (email, user_id);
```

**User Defined Functions:**
```sql
-- Create UDF
CREATE FUNCTION avgState(state tuple<int, bigint>, val int)
CALLED ON NULL INPUT
RETURNS tuple<int, bigint>
LANGUAGE java
AS 'return Tuple.create(state.getInt(0) + 1, state.getLong(1) + val.intValue());';
```

### Limitations

- No joins between tables
- No subqueries
- Limited WHERE clause options
- No transactions across partitions
- No aggregation functions (limited)
- No LIKE operator support

## Interview Questions

### Fundamental Questions

**Q: What is Apache Cassandra and what are its key features?**

A: Apache Cassandra is a distributed NoSQL database designed for handling large amounts of data across multiple servers. Key features include:
- Masterless architecture with no single point of failure
- Linear scalability
- High availability
- Tunable consistency
- Multi-datacenter support
- Column-oriented storage model

**Q: Explain the CAP theorem and how Cassandra fits into it.**

A: CAP theorem states you can only guarantee two of: Consistency, Availability, Partition tolerance. Cassandra is an AP system, prioritizing Availability and Partition tolerance while providing tunable eventual consistency.

**Q: What is the difference between RDBMS and Cassandra?**

A: 
| Aspect | RDBMS | Cassandra |
|--------|-------|-----------|
| Data Model | Relational (tables, rows, columns) | Wide-column store |
| Schema | Fixed schema | Flexible schema |
| ACID | Full ACID compliance | Eventual consistency |
| Scalability | Vertical scaling | Horizontal scaling |
| Joins | Complex joins supported | No joins |
| Transactions | Multi-row transactions | Single-row atomicity |

### Architecture Questions

**Q: Explain Cassandra's ring architecture.**

A: Cassandra uses a ring topology where:
- All nodes are equal (peer-to-peer)
- Each node is assigned a token range
- Data is distributed using consistent hashing
- Nodes communicate via gossip protocol
- No master-slave relationship eliminates single points of failure

**Q: How does consistent hashing work in Cassandra?**

A: Consistent hashing:
- Uses hash function (Murmur3) to map partition keys to token values
- Token space is divided among nodes in the ring
- Data is stored on nodes whose token range includes the data's hash value
- Minimizes data movement when nodes are added/removed
- Ensures even data distribution

**Q: What is the gossip protocol and how does it work?**

A: Gossip protocol is used for:
- Node discovery and failure detection
- Sharing cluster state information
- Propagating schema changes

Process:
- Nodes exchange state information every second
- Each node gossips with 1-3 other nodes
- Information spreads exponentially through the cluster
- Includes node status, load, and schema version

### Performance Questions

**Q: Explain the write path in Cassandra.**

A: Write path involves:
1. Write to commit log for durability
2. Write to memtable in memory
3. When memtable is full, flush to SSTable on disk
4. Coordinate writes to replica nodes based on replication factor
5. Return success when required consistency level is met

**Q: How does Cassandra handle reads?**

A: Read path:
1. Check bloom filters to avoid unnecessary disk reads
2. Check row cache and key cache
3. Read from memtables
4. Read from SSTables if needed
5. Merge results from multiple SSTables using timestamps
6. Coordinate reads from multiple replicas if needed

**Q: What are the different compaction strategies?**

A: 
- **SizeTieredCompactionStrategy (STCS)**: Merges SSTables of similar size, good for write-heavy workloads
- **LeveledCompactionStrategy (LCS)**: Organizes data in levels, good for read-heavy workloads
- **TimeWindowCompactionStrategy (TWCS)**: Groups data by time windows, ideal for time-series data

### Scenario-based Questions

**Q: How would you design a time-series data model for IoT sensors?**

A: 
```sql
CREATE TABLE sensor_data (
    sensor_id UUID,
    year INT,
    month INT,
    timestamp TIMESTAMP,
    temperature DOUBLE,
    humidity DOUBLE,
    PRIMARY KEY ((sensor_id, year, month), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

Design considerations:
- Partition by sensor_id and time buckets to avoid large partitions
- Use clustering key for time-based ordering
- Consider TTL for data expiration
- Use TWCS compaction strategy

**Q: How would you handle a node failure in production?**

A: 
1. **Immediate Response**: Monitor alerts and confirm node status
2. **Assessment**: Check if it's temporary (network) or permanent (hardware)
3. **Temporary Failure**: Wait for node to rejoin, run repair if needed
4. **Permanent Failure**: 
   - Replace node with same token
   - Or remove node and let others handle data
   - Run repair to ensure data consistency
5. **Prevention**: Maintain proper monitoring and backup strategies

**Q: How would you optimize a slow-performing query?**

A: 
1. **Analyze Query Pattern**: Check if it follows data model best practices
2. **Check Data Model**: Ensure efficient partition key selection
3. **Examine Consistency Level**: Use appropriate read consistency
4. **Monitor Resources**: Check CPU, memory, disk I/O
5. **Optimize Configuration**: Tune cache settings, compaction
6. **Consider Indexing**: Secondary indexes or materialized views if appropriate
7. **Data Modeling**: Redesign tables if necessary for query patterns

---

## Contributing

Feel free to contribute to this guide by:
- Adding more interview questions
- Improving explanations
- Adding practical examples
- Fixing any errors or outdated information

## Resources

- [Apache Cassandra Documentation](https://cassandra.apache.org/doc/)
- [DataStax Academy](https://academy.datastax.com/)
- [Cassandra Summit](https://cassandrasummit.org/)
- [Planet Cassandra](https://planetcassandra.org/)

## License

This guide is open source and available under the MIT License.
