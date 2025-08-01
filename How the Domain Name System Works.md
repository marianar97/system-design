# How the Domain Name System Works

## DNS Overview
The Domain Name System (DNS) is a hierarchical and decentralized naming system for computers, services, or other resources connected to the Internet or a private network. It associates various information with domain names assigned to each of the participating entities.

## Key Functions
- **Name Resolution**: Translates human-readable domain names to IP addresses
- **Reverse Lookup**: Translates IP addresses back to domain names
- **Service Discovery**: Locates services within a domain

## DNS Process
1. **User enters URL** in browser (e.g., www.example.com)
2. **Browser checks cache** for recent DNS lookups
3. **OS checks local cache** and hosts file
4. **Query sent to DNS resolver** (usually ISP's DNS server)
5. **Resolver queries root nameserver** (if not cached)
6. **Root server responds** with TLD nameserver info
7. **Resolver queries TLD nameserver** (.com, .org, etc.)
8. **TLD server responds** with authoritative nameserver
9. **Resolver queries authoritative nameserver**
10. **Authoritative server returns** IP address
11. **IP address returned** to browser
12. **Browser connects** to web server using IP address

## DNS Hierarchy
- **Root Level**: Managed by root nameservers (13 globally)
- **Top-Level Domain (TLD)**: .com, .org, .net, country codes
- **Second-Level Domain**: example.com, google.com
- **Subdomain**: www.example.com, mail.example.com

## Performance Considerations
- **Caching**: Multiple levels reduce lookup time
- **TTL (Time To Live)**: Controls cache duration
- **Geographic Distribution**: Servers worldwide for faster responses