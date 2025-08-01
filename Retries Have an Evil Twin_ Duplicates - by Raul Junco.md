# Retries Have an Evil Twin: Duplicates - by Raul Junco

## The Problem with Retries

Retries are a fundamental mechanism for building resilient distributed systems, but they come with an inherent challenge: **duplicates**. When a request fails and gets retried, you might end up processing the same operation multiple times, leading to:

- **Double charging** customers
- **Duplicate notifications** sent to users  
- **Inconsistent data state** across services
- **Resource waste** from redundant processing

## Why Duplicates Happen

### Network Failures
- Request sent successfully but response is lost
- Connection timeout before response received
- Intermittent network partitions

### Service Unavailability
- Downstream service temporarily down
- Load balancer health check failures
- Service overload causing timeouts

### Processing Delays
- Service processing request but taking too long
- Client timeout before service completes
- Async processing delays

## Four Battle-Tested Strategies

### 1. Idempotency Keys

**Concept**: Client generates unique identifier for each operation

```python
class PaymentService:
    def process_payment(self, amount, idempotency_key):
        # Check if we've seen this key before
        existing_result = self.cache.get(idempotency_key)
        if existing_result:
            return existing_result
            
        # Process payment
        result = self.charge_card(amount)
        
        # Cache result with the key
        self.cache.set(idempotency_key, result, ttl=3600)
        
        return result
```

**Pros**:
- Simple to implement
- Works across service boundaries
- Client controls uniqueness

**Cons**:
- Requires client cooperation
- Storage overhead for keys
- Key expiration management

### 2. At-Most-Once Semantics

**Concept**: System guarantees operation executes zero or one times

```python
class OrderProcessor:
    def __init__(self):
        self.processed_orders = set()
        
    def process_order(self, order_id, order_data):
        # Atomic check-and-set operation
        if order_id in self.processed_orders:
            return self.get_existing_result(order_id)
            
        # Process order atomically
        with self.db.transaction():
            self.processed_orders.add(order_id)
            result = self.create_order(order_data)
            self.save_result(order_id, result)
            
        return result
```

**Implementation Techniques**:
- **Database constraints**: Unique indexes prevent duplicates
- **Conditional writes**: Only write if condition met
- **Compare-and-swap**: Atomic operations

### 3. Exactly-Once Processing

**Concept**: Combine deduplication with transactional guarantees

```python
class MessageProcessor:
    def process_message(self, message_id, message_data):
        with self.db.transaction():
            # Check if already processed
            if self.is_message_processed(message_id):
                return self.get_processing_result(message_id)
                
            # Process message
            result = self.handle_message(message_data)
            
            # Mark as processed atomically
            self.mark_message_processed(message_id, result)
            
            return result
            
    def is_message_processed(self, message_id):
        return self.db.exists("processed_messages", message_id)
        
    def mark_message_processed(self, message_id, result):
        self.db.insert("processed_messages", {
            "message_id": message_id,
            "result": result,
            "processed_at": datetime.now()
        })
```

**Key Requirements**:
- Transactional storage
- Atomic operations
- Consistent state management

### 4. Client-Side Deduplication

**Concept**: Client tracks requests and handles duplicates

```python
class APIClient:
    def __init__(self):
        self.pending_requests = {}
        self.completed_requests = {}
        
    def make_request(self, request_id, request_data):
        # Check if request already completed
        if request_id in self.completed_requests:
            return self.completed_requests[request_id]
            
        # Check if request already in flight
        if request_id in self.pending_requests:
            return self.pending_requests[request_id].get()
            
        # Make new request
        future = self.executor.submit(self.send_request, request_data)
        self.pending_requests[request_id] = future
        
        try:
            result = future.result(timeout=30)
            self.completed_requests[request_id] = result
            return result
        finally:
            self.pending_requests.pop(request_id, None)
```

## Implementation Patterns

### Database-Level Deduplication

```sql
-- Use unique constraints
CREATE TABLE payments (
    id UUID PRIMARY KEY,
    idempotency_key VARCHAR(255) UNIQUE NOT NULL,
    amount DECIMAL(10,2),
    status VARCHAR(50),
    created_at TIMESTAMP
);

-- Prevent duplicate processing
INSERT INTO payments (id, idempotency_key, amount, status)
VALUES (gen_random_uuid(), 'pay_123', 100.00, 'completed')
ON CONFLICT (idempotency_key) DO NOTHING;
```

### Message Queue Deduplication

```python
class DeduplicatingQueue:
    def __init__(self):
        self.message_hashes = BloomFilter(capacity=1000000, error_rate=0.1)
        self.recent_messages = LRUCache(maxsize=10000)
        
    def enqueue(self, message):
        message_hash = self.hash_message(message)
        
        # Quick bloom filter check
        if message_hash in self.message_hashes:
            # More expensive exact check
            if message_hash in self.recent_messages:
                return False  # Duplicate detected
                
        # Add to tracking structures
        self.message_hashes.add(message_hash)
        self.recent_messages[message_hash] = True
        
        # Enqueue message
        self.queue.put(message)
        return True
```

### Distributed Deduplication

```python
class DistributedDeduplicator:
    def __init__(self, redis_client):
        self.redis = redis_client
        
    def is_duplicate(self, operation_id, ttl=3600):
        # Use Redis SET with NX (not exists) option
        result = self.redis.set(
            f"dedup:{operation_id}", 
            "1", 
            nx=True,  # Only set if key doesn't exist
            ex=ttl    # Expire after TTL seconds
        )
        
        return result is None  # None means key already existed
        
    def process_if_unique(self, operation_id, operation_func):
        if not self.is_duplicate(operation_id):
            return operation_func()
        else:
            # Return cached result or indicate duplicate
            return self.get_cached_result(operation_id)
```

## Choosing the Right Strategy

### Decision Matrix

| Strategy | Complexity | Performance | Reliability | Use Case |
|----------|------------|-------------|-------------|----------|
| Idempotency Keys | Low | High | High | REST APIs, payments |
| At-Most-Once | Medium | Medium | High | Critical operations |
| Exactly-Once | High | Low | Very High | Financial transactions |
| Client-Side | Low | High | Medium | Simple APIs |

### Considerations

**For HTTP APIs**:
- Use idempotency keys for POST requests
- Leverage HTTP method semantics (GET, PUT are naturally idempotent)
- Implement server-side tracking

**For Message Processing**:
- Exactly-once processing for critical data
- At-least-once with deduplication for high throughput
- Client-side deduplication for simple cases

**For Payments/Financial**:
- Always use exactly-once semantics
- Multiple layers of deduplication
- Audit trails and reconciliation

## Best Practices

### Key Generation
```python
# Good: Include relevant context
idempotency_key = f"payment_{user_id}_{amount}_{timestamp}"

# Better: Use UUIDs for uniqueness
idempotency_key = str(uuid.uuid4())

# Best: Client-generated with semantic meaning
idempotency_key = f"order_{order_id}_retry_{retry_count}"
```

### TTL Management
```python
class TTLManager:
    def get_ttl(self, operation_type):
        ttls = {
            'payment': 86400,      # 24 hours
            'notification': 3600,   # 1 hour
            'data_sync': 300       # 5 minutes
        }
        return ttls.get(operation_type, 3600)
```

### Error Handling
```python
def safe_deduplication(self, key, operation):
    try:
        if self.is_duplicate(key):
            return self.get_cached_result(key)
    except DeduplicationError:
        # Fallback: allow operation but log warning
        logger.warning(f"Deduplication check failed for {key}")
        
    return operation()
```

## Monitoring and Observability

### Key Metrics
- **Duplicate detection rate**: How often duplicates are caught
- **False positive rate**: Legitimate requests marked as duplicates
- **Deduplication cache hit rate**: Efficiency of caching layer
- **Processing time impact**: Overhead of deduplication logic

### Alerting
```python
class DeduplicationMonitor:
    def track_duplicate_rate(self, operation_type):
        rate = self.get_duplicate_rate(operation_type)
        
        if rate > self.thresholds[operation_type]:
            self.alert_manager.send_alert(
                f"High duplicate rate for {operation_type}: {rate}%"
            )
            
    def track_cache_performance(self):
        hit_rate = self.cache.get_hit_rate()
        
        if hit_rate < 0.8:  # Below 80%
            self.alert_manager.send_alert(
                f"Low deduplication cache hit rate: {hit_rate}"
            )
```

## Conclusion

Handling duplicates is not optional in distributed systems—it's a fundamental requirement. The key is choosing the right strategy based on your specific requirements:

- **High performance, simple cases**: Client-side deduplication
- **API reliability**: Idempotency keys  
- **Critical operations**: At-most-once semantics
- **Financial systems**: Exactly-once processing

Remember: **The cost of handling duplicates is always less than the cost of not handling them.**