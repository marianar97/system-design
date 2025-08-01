# Introduction to Domain Name System (DNS)

## What is DNS?
DNS (Domain Name System) is a hierarchical and decentralized naming system that translates human-readable domain names into IP addresses that computers use to identify each other on the network.

## Why DNS is Needed
- **Human-friendly**: People remember names better than numbers
- **Flexibility**: IP addresses can change without affecting users
- **Load Distribution**: Multiple IP addresses for same domain
- **Service Organization**: Different services on different servers

## DNS Hierarchy

### Root Level (.)
- **13 root servers** worldwide (A through M)
- **Managed by different organizations**
- **Know where to find TLD servers**

### Top-Level Domains (TLD)
- **Generic TLDs**: .com, .org, .net, .edu, .gov
- **Country Code TLDs**: .uk, .ca, .de, .jp
- **New gTLDs**: .tech, .blog, .app

### Second-Level Domains
- **Registered by individuals/organizations**
- **Examples**: google.com, facebook.com
- **Managed by domain registrars**

### Subdomains
- **Created by domain owners**
- **Examples**: www.google.com, mail.google.com
- **Can have multiple levels**: blog.news.example.com

## DNS Record Types

### A Record
- **Maps domain to IPv4 address**
- **Example**: example.com → 192.0.2.1

### AAAA Record
- **Maps domain to IPv6 address**
- **Example**: example.com → 2001:db8::1

### CNAME Record
- **Creates alias for another domain**
- **Example**: www.example.com → example.com

### MX Record
- **Specifies mail exchange servers**
- **Includes priority values**
- **Example**: example.com MX 10 mail.example.com

### NS Record
- **Delegates subdomain to nameservers**
- **Example**: subdomain.example.com NS ns1.provider.com

### TXT Record
- **Stores text information**
- **Used for verification, SPF, DKIM**

### PTR Record
- **Reverse DNS lookup**
- **Maps IP address to domain name**

## DNS Resolution Process

### 1. Recursive Query
- **Client asks resolver** for complete answer
- **Resolver does all the work**
- **Returns final result to client**

### 2. Iterative Query
- **Server responds with referral** to another server
- **Client must query next server**
- **Process continues until resolved**

### 3. Caching
- **Results cached at multiple levels**:
  - Browser cache
  - OS cache
  - Router cache
  - ISP resolver cache
- **TTL (Time To Live)** controls cache duration
- **Reduces query load and improves performance**

## DNS Servers

### Recursive Resolver
- **Receives queries from clients**
- **Performs iterative queries to find answer**
- **Caches results for future queries**
- **Examples**: ISP DNS, Google (8.8.8.8), Cloudflare (1.1.1.1)

### Authoritative Nameserver
- **Holds DNS records for domain**
- **Provides definitive answers**
- **No caching involved**
- **Examples**: Domain registrar servers, self-hosted

### Root Nameserver
- **Top of DNS hierarchy**
- **13 logical servers (A-M root)**
- **Operated by different organizations**
- **Respond with TLD server information**

## DNS Security

### DNS Spoofing/Poisoning
- **Attacker provides false DNS responses**
- **Redirects traffic to malicious servers**
- **Mitigation**: DNSSEC, secure resolvers

### DNS Hijacking
- **Unauthorized changes to DNS settings**
- **Domain registration theft**
- **Mitigation**: Registry lock, 2FA

### DNSSEC (DNS Security Extensions)
- **Cryptographic signatures for DNS records**
- **Ensures authenticity and integrity**
- **Prevents spoofing and tampering**
- **Chain of trust from root to domain**

## Performance Optimization

### Geographic Distribution
- **Anycast routing**
- **Servers in multiple locations**
- **Users connect to nearest server**

### Load Balancing
- **Multiple A records for same domain**
- **Round-robin or weighted responses**
- **Health checking of servers**

### TTL Optimization
- **Long TTL**: Reduces queries, slower updates
- **Short TTL**: Fresh data, more queries
- **Balance based on update frequency**

## DNS in System Design

### Considerations
- **Single point of failure risk**
- **Propagation delays for changes**
- **Caching complexities**
- **Security vulnerabilities**

### Best Practices
- **Use multiple nameservers**
- **Implement health checks**
- **Plan for DNS failover**
- **Monitor DNS performance**
- **Consider DNS-based load balancing**