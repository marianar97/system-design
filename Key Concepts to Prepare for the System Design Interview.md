# Key Concepts to Prepare for the System Design Interview

## CAP Theorem
- **Consistency**: All nodes see the same data simultaneously
- **Availability**: System remains operational 
- **Partition tolerance**: System continues despite network failures
- You can only guarantee 2 out of 3 properties

## PACELC Theorem
Extension of CAP theorem:
- **If Partition** occurs: choose between **Availability** or **Consistency**
- **Else** (no partition): choose between **Latency** or **Consistency**

## Heartbeat
- Mechanism to detect failures in distributed systems
- Nodes send periodic signals to indicate they're alive
- If heartbeat stops, node is considered failed

## System Design Fundamentals
- **Scalability**: Ability to handle increased load
- **Reliability**: System continues to work correctly
- **Availability**: System remains operational over time
- **Consistency**: All nodes see the same data
- **Load Balancing**: Distribute requests across multiple servers
- **Caching**: Store frequently accessed data for faster retrieval
- **Data Partitioning**: Split data across multiple databases
- **Replication**: Keep copies of data on multiple nodes

## Key Architectural Patterns
- **Microservices**: Break system into small, independent services
- **Event-Driven Architecture**: Components communicate through events
- **CQRS**: Separate read and write operations
- **Database Sharding**: Horizontally partition database
- **Circuit Breaker**: Prevent cascading failures

## Performance Metrics
- **Throughput**: Requests handled per unit time
- **Latency**: Time to process a single request
- **Response Time**: Time from request to response
- **Availability**: Percentage of time system is operational