# High-level Design of WhatsApp

## System Requirements

### Functional Requirements
- **Send and receive messages** in real-time
- **Support 1-on-1 and group conversations**
- **Message delivery confirmation** (sent, delivered, read)
- **Online/offline user status**
- **Support multimedia messages** (images, videos, documents)

### Non-functional Requirements
- **High availability**: 99.9% uptime
- **Low latency**: Messages delivered within seconds
- **Scalability**: Support billions of users
- **Consistency**: Messages delivered in order
- **Security**: End-to-end encryption

## Scale Estimations

### Users and Traffic
- **2 billion monthly active users**
- **100 billion messages per day**
- **Average message size**: 100 bytes
- **Peak traffic**: 3-4x average during peak hours

### Storage Requirements
- **Daily storage**: 100B messages × 100 bytes = 10 TB/day
- **Monthly storage**: 10 TB × 30 = 300 TB/month
- **With media**: 10x increase = 3 PB/month

## High-level Architecture

### Client Applications
- **Mobile apps** (iOS, Android)
- **Web client**
- **Desktop applications**

### Load Balancer
- **Distributes incoming connections**
- **SSL termination**
- **Health checking of backend services**

### WebSocket Servers
- **Maintain persistent connections** with clients
- **Handle real-time message routing**
- **Connection pooling and management**

### API Gateway
- **Authentication and authorization**
- **Rate limiting**
- **Request routing to microservices**

### Message Service
- **Core messaging logic**
- **Message validation and processing**
- **Delivery status tracking**

### User Service
- **User profile management**
- **Friend/contact lists**
- **Online status management**

### Notification Service
- **Push notifications for offline users**
- **Email notifications**
- **SMS fallback**

## WebSocket Communication Flow

### Connection Establishment
1. **Client connects** to load balancer
2. **Load balancer routes** to available WebSocket server
3. **WebSocket server authenticates** user
4. **Connection registered** in connection registry

### Message Flow
1. **Sender sends message** via WebSocket
2. **WebSocket server receives** and validates message
3. **Message sent to Message Service** for processing
4. **Message Service determines** recipient's server
5. **Message routed** to recipient's WebSocket server
6. **Message delivered** to recipient if online
7. **Delivery confirmation** sent back to sender

## API Design

### Send Message
```
POST /api/v1/messages
{
  "recipient_id": "user123",
  "message_type": "text",
  "content": "Hello, how are you?",
  "timestamp": 1640995200
}
```

### Get Messages
```
GET /api/v1/messages?conversation_id=conv123&limit=50&offset=0
```

### Update Message Status
```
PUT /api/v1/messages/{message_id}/status
{
  "status": "delivered",
  "timestamp": 1640995300
}
```

## Database Design

### Message Storage
- **Sharding by conversation_id**
- **Time-based partitioning**
- **NoSQL database** (Cassandra, HBase)

### User Data
- **Relational database** for user profiles
- **Redis cache** for online status
- **Consistent hashing** for distribution

### Connection Registry
- **In-memory store** (Redis)
- **Maps user_id to WebSocket server**
- **TTL for connection cleanup**

## Caching Strategy

### User Cache
- **Active user profiles**
- **Friend lists and contacts**
- **Recent conversation metadata**

### Message Cache
- **Recent messages per conversation**
- **LRU eviction policy**
- **Write-through caching**

## Security Considerations

### Authentication
- **OAuth 2.0 / JWT tokens**
- **Phone number verification**
- **Device registration**

### Message Security
- **End-to-end encryption**
- **Message integrity checks**
- **Forward secrecy**

### Network Security
- **TLS/SSL for all connections**
- **DDoS protection**
- **Rate limiting per user**

## Monitoring and Reliability

### Health Checks
- **WebSocket server health**
- **Database connectivity**
- **Service dependencies**

### Metrics
- **Message delivery rates**
- **Connection counts**
- **Latency measurements**
- **Error rates**

### Disaster Recovery
- **Multi-region deployment**
- **Database replication**
- **Automated failover**

## Scalability Considerations

### Horizontal Scaling
- **Stateless WebSocket servers**
- **Database sharding**
- **Load balancer clusters**

### Performance Optimization
- **Connection pooling**
- **Message batching**
- **Compression algorithms**
- **CDN for media files**

## Technology Stack

### Backend
- **Java/Go** for WebSocket servers
- **Python/Node.js** for API services
- **Kafka** for message queuing
- **Cassandra/HBase** for message storage

### Infrastructure
- **Kubernetes** for container orchestration
- **Docker** for containerization
- **AWS/GCP** for cloud infrastructure
- **Redis** for caching and session storage