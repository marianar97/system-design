# Examples of Resource Estimation

## Twitter-like Service Example

### Given Requirements
- 300 million monthly active users
- 50% of users use Twitter daily
- Users post 2 tweets per day on average
- 10% of tweets contain media
- Data is stored for 5 years

### Estimations

#### Traffic Estimates
- **Daily Active Users (DAU)**: 300M × 50% = 150M
- **Tweets per second**: 150M × 2 tweets / 24 hours / 3600 seconds = ~3,500 tweets/sec
- **Peak load**: 2 × 3,500 = 7,000 tweets/sec
- **Read QPS**: Assuming 1:100 write/read ratio = 350,000 reads/sec

#### Storage Estimates
- **Average tweet size**: 280 characters = 280 bytes
- **Media per tweet**: 10% of tweets have media, average 1MB
- **Daily storage**: 
  - Text: 150M × 2 × 280 bytes = 84 GB
  - Media: 150M × 2 × 0.1 × 1MB = 30 TB
- **Total daily**: ~30 TB
- **5-year storage**: 30 TB × 365 × 5 = ~55 PB

#### Bandwidth Estimates
- **Write bandwidth**: 30 TB / 24 hours / 3600 seconds = ~350 MB/s
- **Read bandwidth**: Assuming 1:100 ratio = ~35 GB/s

#### Server Estimates
- **For writes**: 7,000 tweets/sec ÷ 1,000 QPS per server = 7 servers
- **For reads**: 350,000 reads/sec ÷ 10,000 QPS per server = 35 servers
- **With redundancy**: 2-3x more servers = ~100-150 servers

#### Memory/Cache Estimates
- **Follow 80-20 rule**: 20% of tweets generate 80% of traffic
- **Cache hot tweets**: 20% of daily tweets = 0.2 × 150M × 2 = 60M tweets
- **Memory needed**: 60M × 280 bytes = ~17 GB
- **Add media metadata**: ~20 GB total per cache server

## Additional Considerations
- **Database sharding**: Based on user_id or tweet_id
- **Content Delivery Network (CDN)**: For media files
- **Load balancers**: Multiple layers for traffic distribution
- **Monitoring and logging**: Additional 10-20% overhead
- **Backup and disaster recovery**: 2-3x storage requirements