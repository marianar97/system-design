# Availability

## Definition
Availability is the percentage of time a system is operational and accessible when required for use.

## Availability Levels (Nines)
- **90% (1 nine)**: 36.5 days downtime per year
- **99% (2 nines)**: 3.65 days downtime per year  
- **99.9% (3 nines)**: 8.76 hours downtime per year
- **99.99% (4 nines)**: 52.56 minutes downtime per year
- **99.999% (5 nines)**: 5.26 minutes downtime per year

## Service Provider Considerations
- **SLA (Service Level Agreement)**: Contractual commitment to availability
- **SLO (Service Level Objective)**: Internal target for service performance
- **SLI (Service Level Indicator)**: Actual measurement of service performance

## Factors Affecting Availability
- Hardware failures
- Software bugs
- Network issues
- Human errors
- Natural disasters
- Maintenance windows

## Strategies to Improve Availability
- **Redundancy**: Multiple instances of critical components
- **Failover**: Automatic switching to backup systems
- **Load Balancing**: Distribute traffic across multiple servers
- **Monitoring**: Continuous health checks and alerting
- **Graceful Degradation**: Partial functionality during failures
- **Circuit Breakers**: Prevent cascading failures

## Availability vs Other Qualities
- **Availability vs Consistency**: CAP theorem trade-offs
- **Availability vs Performance**: Sometimes need to sacrifice speed for uptime
- **Availability vs Cost**: Higher availability requires more resources