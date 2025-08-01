# Detailed Design of WhatsApp

## Deep Dive into Core Components

### WebSocket Server Architecture

#### Connection Management
- **Connection Pool**: Manages active WebSocket connections
- **User Session Store**: Maps user_id to connection details
- **Heartbeat Mechanism**: Detects dead connections
- **Connection Cleanup**: Removes stale connections

#### Message Routing
```python
class MessageRouter:
    def route_message(self, message):
        recipient_id = message.recipient_id
        connection = self.get_user_connection(recipient_id)
        
        if connection:
            # User is online, send directly
            connection.send(message)
            return "delivered"
        else:
            # User offline, queue for later delivery
            self.message_queue.enqueue(message)
            return "queued"
```

#### Load Balancing Strategy
- **Consistent Hashing**: Route users to specific servers
- **Session Affinity**: Keep user on same server during session
- **Failover Mechanism**: Redirect connections on server failure

### Message Service Deep Dive

#### Message Processing Pipeline
1. **Validation**: Check message format and permissions
2. **Encryption**: Apply end-to-end encryption
3. **Storage**: Persist message to database
4. **Delivery**: Route to recipient's server
5. **Acknowledgment**: Send delivery confirmation

#### Message Storage Schema
```sql
-- Cassandra/HBase schema
CREATE TABLE messages (
    conversation_id UUID,
    message_id UUID,
    sender_id UUID,
    recipient_id UUID,
    message_type VARCHAR,
    content TEXT,
    timestamp BIGINT,
    status VARCHAR,
    PRIMARY KEY (conversation_id, timestamp, message_id)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

#### Delivery Guarantees
- **At-least-once delivery**: Messages not lost
- **Idempotency**: Duplicate detection
- **Ordering**: FIFO per conversation

### Group Messaging Architecture

#### Group Management
```python
class GroupService:
    def send_group_message(self, group_id, message):
        members = self.get_group_members(group_id)
        
        # Fan-out to all members
        for member_id in members:
            individual_message = self.create_individual_message(
                message, member_id
            )
            self.message_service.send_message(individual_message)
```

#### Group Storage
```sql
CREATE TABLE groups (
    group_id UUID PRIMARY KEY,
    group_name VARCHAR,
    created_by UUID,
    created_at TIMESTAMP,
    max_members INT
);

CREATE TABLE group_members (
    group_id UUID,
    user_id UUID,
    joined_at TIMESTAMP,
    role VARCHAR,
    PRIMARY KEY (group_id, user_id)
);
```

#### Large Group Optimization
- **Message Broadcasting**: Efficient fan-out mechanisms
- **Member Pagination**: Load members in batches
- **Admin Controls**: Restrict who can send messages

### User Service Implementation

#### User Profile Management
```python
class UserService:
    def get_user_profile(self, user_id):
        # Check cache first
        profile = self.cache.get(f"user:{user_id}")
        if profile:
            return profile
            
        # Fallback to database
        profile = self.db.get_user(user_id)
        self.cache.set(f"user:{user_id}", profile, ttl=3600)
        return profile
```

#### Online Status Tracking
```python
class PresenceService:
    def update_user_status(self, user_id, status):
        # Update in Redis with TTL
        self.redis.setex(f"status:{user_id}", 300, status)
        
        # Notify contacts about status change
        contacts = self.get_user_contacts(user_id)
        for contact_id in contacts:
            self.notify_status_change(contact_id, user_id, status)
```

## Advanced Features Implementation

### Message Encryption
```python
class E2EEncryption:
    def encrypt_message(self, message, recipient_public_key):
        # Generate ephemeral key pair
        ephemeral_private, ephemeral_public = self.generate_key_pair()
        
        # Derive shared secret
        shared_secret = self.ecdh(ephemeral_private, recipient_public_key)
        
        # Encrypt message
        encrypted_content = self.aes_encrypt(message.content, shared_secret)
        
        return {
            'ephemeral_public_key': ephemeral_public,
            'encrypted_content': encrypted_content,
            'mac': self.hmac(encrypted_content, shared_secret)
        }
```

### Media Message Handling
```python
class MediaService:
    def upload_media(self, file_data, user_id):
        # Generate unique file ID
        file_id = self.generate_file_id()
        
        # Upload to cloud storage (S3, GCS)
        storage_url = self.cloud_storage.upload(file_data, file_id)
        
        # Generate thumbnail for images/videos
        if file_data.type in ['image', 'video']:
            thumbnail = self.generate_thumbnail(file_data)
            thumbnail_url = self.cloud_storage.upload(thumbnail, f"{file_id}_thumb")
        
        return {
            'file_id': file_id,
            'url': storage_url,
            'thumbnail_url': thumbnail_url,
            'size': file_data.size,
            'type': file_data.type
        }
```

### Message Synchronization
```python
class SyncService:
    def sync_messages(self, user_id, device_id, last_sync_timestamp):
        # Get messages since last sync
        messages = self.db.get_messages_since(user_id, last_sync_timestamp)
        
        # Batch messages for efficient transfer
        batches = self.batch_messages(messages, batch_size=100)
        
        for batch in batches:
            self.send_sync_batch(device_id, batch)
            
        # Update sync timestamp
        self.update_last_sync(user_id, device_id, current_timestamp())
```

## Database Sharding Strategy

### Message Sharding
- **Shard Key**: conversation_id hash
- **Benefits**: Related messages on same shard
- **Rebalancing**: Consistent hashing for smooth migration

### User Data Sharding
- **Shard Key**: user_id hash
- **Cross-shard queries**: For friend lists across shards
- **Caching**: Reduce cross-shard queries

## Caching Architecture

### Multi-level Caching
```python
class CacheManager:
    def __init__(self):
        self.l1_cache = LRUCache(size=10000)  # Local cache
        self.l2_cache = RedisCluster()        # Distributed cache
        self.database = CassandraCluster()    # Persistent storage
    
    def get(self, key):
        # Try L1 cache first
        value = self.l1_cache.get(key)
        if value:
            return value
            
        # Try L2 cache
        value = self.l2_cache.get(key)
        if value:
            self.l1_cache.put(key, value)
            return value
            
        # Fallback to database
        value = self.database.get(key)
        if value:
            self.l2_cache.put(key, value)
            self.l1_cache.put(key, value)
            
        return value
```

### Cache Invalidation
- **Write-through**: Update cache when writing to database
- **TTL-based**: Automatic expiration for stale data
- **Event-driven**: Invalidate on specific events

## Performance Optimizations

### Connection Pooling
```python
class ConnectionPool:
    def __init__(self, max_connections=1000):
        self.pool = Queue(maxsize=max_connections)
        self.active_connections = {}
        
    def get_connection(self, user_id):
        if user_id in self.active_connections:
            return self.active_connections[user_id]
            
        if not self.pool.empty():
            connection = self.pool.get()
            self.active_connections[user_id] = connection
            return connection
            
        # Create new connection if pool not full
        if len(self.active_connections) < self.max_connections:
            connection = self.create_connection()
            self.active_connections[user_id] = connection
            return connection
            
        return None  # Pool exhausted
```

### Message Batching
```python
class MessageBatcher:
    def __init__(self, batch_size=50, flush_interval=100):
        self.batch_size = batch_size
        self.flush_interval = flush_interval
        self.pending_messages = []
        self.last_flush = time.time()
        
    def add_message(self, message):
        self.pending_messages.append(message)
        
        if (len(self.pending_messages) >= self.batch_size or 
            time.time() - self.last_flush > self.flush_interval):
            self.flush_batch()
            
    def flush_batch(self):
        if self.pending_messages:
            self.process_batch(self.pending_messages)
            self.pending_messages = []
            self.last_flush = time.time()
```

## Monitoring and Observability

### Key Metrics
- **Message delivery latency**: P95, P99 percentiles
- **Connection count**: Active WebSocket connections
- **Throughput**: Messages per second
- **Error rates**: Failed deliveries, connection drops
- **Cache hit rates**: L1/L2 cache performance

### Alerting System
```python
class AlertManager:
    def check_message_latency(self):
        latency_p99 = self.metrics.get_p99_latency()
        if latency_p99 > self.SLA_LATENCY:
            self.send_alert(
                severity="high",
                message=f"Message latency P99: {latency_p99}ms exceeds SLA"
            )
    
    def check_error_rate(self):
        error_rate = self.metrics.get_error_rate()
        if error_rate > self.MAX_ERROR_RATE:
            self.send_alert(
                severity="critical",
                message=f"Error rate: {error_rate}% exceeds threshold"
            )
```

### Distributed Tracing
- **Trace message flow**: From sender to recipient
- **Identify bottlenecks**: Slow database queries, network issues
- **Service dependencies**: Map inter-service communications

## Disaster Recovery

### Multi-region Setup
- **Active-Active**: Both regions serve traffic
- **Data Replication**: Async replication between regions
- **Failover**: Automatic traffic redirection

### Backup Strategy
- **Message Backup**: Daily incremental backups
- **Point-in-time Recovery**: Restore to specific timestamp
- **Cross-region Backup**: Backup to different geographic location