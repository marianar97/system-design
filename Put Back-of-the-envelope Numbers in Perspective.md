# Put Back-of-the-envelope Numbers in Perspective

## Server Specifications (Typical)
- **CPU**: 16-32 cores
- **RAM**: 64-256 GB
- **Storage**: 1-10 TB SSD
- **Network**: 1-10 Gbps

## Latency Numbers Every Programmer Should Know
- **L1 cache reference**: 0.5 ns
- **Branch mispredict**: 5 ns
- **L2 cache reference**: 7 ns
- **Mutex lock/unlock**: 25 ns
- **Main memory reference**: 100 ns
- **Compress 1K bytes with Zippy**: 10,000 ns = 10 μs
- **Send 1 KB over 1 Gbps network**: 10,000 ns = 10 μs
- **Read 4 KB randomly from SSD**: 150,000 ns = 150 μs
- **Read 1 MB sequentially from memory**: 250,000 ns = 250 μs
- **Round trip within same datacenter**: 500,000 ns = 500 μs
- **Read 1 MB sequentially from SSD**: 1,000,000 ns = 1 ms
- **Disk seek**: 10,000,000 ns = 10 ms
- **Read 1 MB sequentially from network**: 10,000,000 ns = 10 ms
- **Read 1 MB sequentially from disk**: 30,000,000 ns = 30 ms
- **Send packet CA→Netherlands→CA**: 150,000,000 ns = 150 ms

## Throughput Numbers
- **QPS (Queries Per Second)**:
  - Single server: 1,000-10,000 QPS
  - With caching: 10,000-100,000 QPS
  - Distributed system: 100,000+ QPS

## Storage Calculations
- **1 character = 1 byte**
- **1 KB = 1,000 bytes**
- **1 MB = 1,000 KB = 1,000,000 bytes**
- **1 GB = 1,000 MB**
- **1 TB = 1,000 GB**

## Bandwidth Calculations
- **1 Mbps = 125 KB/s**
- **1 Gbps = 125 MB/s**
- **Typical home internet**: 25-100 Mbps
- **Datacenter connection**: 1-10 Gbps

## Rule of Thumb Estimates
- **Daily Active Users**: 20% of Monthly Active Users
- **Peak QPS**: 2-3x average QPS
- **Read/Write Ratio**: Usually 100:1 to 1000:1
- **Data Growth**: Plan for 10x growth in 5 years