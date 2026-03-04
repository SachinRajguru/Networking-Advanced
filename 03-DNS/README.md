
# DNS (Domain Name System)

---

## 1. What is DNS?

**DNS** (Domain Name System) is a hierarchical, distributed database system that translates domain names into IP addresses and vice versa. Without DNS, we would need to remember IP addresses for every website we want to visit.

### DNS as the Internet Phonebook

| Human-Readable | Computer-Readable |
|----------------|-------------------|
| `google.com` | `142.250.185.46` |
| `example.com` | `93.184.216.34` |
| `github.com` | `140.82.121.3` |

---

## 2. Why DNS Was Created

### The Problem Before DNS (pre-1983)

| Issue | Description |
|-------|-------------|
| **HOSTS.TXT file** | Single text file maintained by Stanford Research Institute |
| **Downloaded periodically** | All hosts on the internet downloaded the file |
| **File became too large** | As internet grew, file became unmanageable |
| **Updates too frequent** | Network congestion from constant downloads |
| **No name uniqueness** | No guarantee of unique names |
| **No scalability** | Could not handle billions of hosts |

### The DNS Solution

| Feature | Benefit |
|---------|---------|
| **Distributed** | No single point of failure |
| **Hierarchical** | Organized tree structure |
| **Cached** | Faster responses, reduced load |
| **Scalable** | Can handle billions of queries |
| **Delegated** | Different organizations manage different parts |

---

## 3. How DNS Fundamentally Works

DNS operates on a **query-response** model:

```
1. Application needs IP: Browser wants to visit example.com
2. Query sent to resolver: "What's the IP for example.com?"
3. Resolver checks cache: Recently looked up?
4. If not cached, recursive resolution begins
5. IP returned to application
6. Result cached for future use
```

### DNS as a Distributed Database

DNS is not a single server but millions of servers worldwide:

| Server Type | Purpose |
|-------------|---------|
| **Root servers** | 13 root server clusters (hundreds of actual servers) |
| **TLD servers** | Servers for .com, .org, .net, etc. |
| **Authoritative servers** | Servers that know specific domains |
| **Recursive resolvers** | Servers that do the lookup work for clients |

**Benefits of Distribution**:
- **Redundancy**: No single point of failure
- **Performance**: Queries answered locally when possible
- **Scalability**: Load distributed across many servers

---

## 4. How DNS Works (Simple Flow)

```
1. You type: www.example.com in browser
2. Browser asks DNS resolver: "What's the IP for example.com?"
3. DNS resolver checks its cache
4. If not cached, resolver asks root servers → TLD servers → authoritative server
5. Gets IP: 93.184.216.34
6. Browser connects to that IP
```

---

## 5. DNS Hierarchy

```
                        . (Root)
                         |
        ┌────────────────┼────────────────┐
       .com            .org             .net
        |
    example.com
        |
   ┌────┴────┐
  www      api
```

---

## 6. Types of DNS Records

| Record Type | Purpose | Example |
|-------------|---------|---------|
| **A** | Maps domain to IPv4 | `example.com → 93.184.216.34` |
| **AAAA** | Maps domain to IPv6 | `example.com → 2606:2800:220:1:248:1893:25c8:1946` |
| **CNAME** | Alias to another domain | `www.example.com → example.com` |
| **MX** | Mail server | `example.com → mail.example.com` |
| **TXT** | Text records (verification, SPF) | Used for domain verification |
| **NS** | Nameserver records | Points to DNS servers |
| **PTR** | Reverse DNS (IP to domain) | `34.216.184.93 → example.com` |
| **SOA** | Start of Authority | Zone management information |
| **SRV** | Service records | Service location information |

---

## 7. Real-World Examples

### Example 1: Basic Website Setup

```
example.com.        A       93.184.216.34
www.example.com.    CNAME   example.com.
```

Both `example.com` and `www.example.com` point to the same server.

### Example 2: Load Balanced Application

```
example.com.    A    10.0.1.5
example.com.    A    10.0.1.6
example.com.    A    10.0.1.7
```

DNS returns all IPs (Round-robin DNS load balancing).

### Example 3: Microservices in Cloud

```
api.example.com      A       54.123.45.67
admin.example.com    A       54.123.45.68
cdn.example.com      CNAME   d111111abcdef8.cloudfront.net
```

---

## 8. DNS Resolution Process (Detailed)

### The Complete Journey of a DNS Query

When you type `www.example.com` in your browser, here's the detailed process:

### Step 1: Browser Cache Check

```
Browser: "Do I already know example.com's IP?"
Cache: "Yes, it's 93.184.216.34" → DONE (fastest)
OR
Cache: "No, I don't have it" → Continue to Step 2
```

### Step 2: Operating System Cache

```
OS: "Do I have example.com cached?"
Cache: "Yes, here's the IP" → DONE
OR
Cache: "No" → Continue to Step 3
```

### Step 3: Recursive Resolver Query

```
Your computer → Recursive Resolver (usually your ISP or 8.8.8.8)
"What's the IP for www.example.com?"
```

### Step 4: Recursive Resolution Begins

The resolver doesn't know the answer, so it asks other DNS servers:

```
Step 4a: Query Root Server
Resolver → Root Server: "Where can I find .com?"
Root Server → Resolver: "Ask the .com TLD servers at these IPs"

Step 4b: Query TLD Server
Resolver → .com TLD Server: "Where can I find example.com?"
TLD Server → Resolver: "Ask example.com's nameservers at these IPs"

Step 4c: Query Authoritative Server
Resolver → example.com Nameserver: "What's www.example.com?"
Authoritative Server → Resolver: "It's 93.184.216.34"

Step 4d: Return to Client
Resolver → Your Computer: "www.example.com is 93.184.216.34"
Resolver also caches this for future queries
```

### Step 5: Connection

```
Browser now connects to 93.184.216.34
```

---

## 9. Types of DNS Queries

### 1. Recursive Query

```
Client → Resolver: "Give me the answer or an error"
Resolver must do all the work and return final answer
Most common type from clients to resolvers
```

### 2. Iterative Query

```
Resolver → Server: "What do you know about example.com?"
Server: "I don't know, but try this server instead"
Resolver follows referrals until it gets answer
Used between resolvers and DNS servers
```

### 3. Non-Recursive Query

```
Resolver → Server: "What's example.com?"
Server: "I have it cached, here you go"
No additional queries needed
```

### DNS Query Types

| Query Type | Description |
|------------|-------------|
| **A record query** | "What's the IPv4 address?" |
| **AAAA record query** | "What's the IPv6 address?" |
| **MX record query** | "Where should I send mail?" |
| **NS record query** | "Who are the nameservers?" |
| **ANY query** | "Give me all records" (often blocked now) |

---

## 10. DNS Caching

DNS responses are cached at multiple levels to improve performance and reduce load on DNS infrastructure.

### Cache Hierarchy (Top to Bottom)

| Cache Level | Duration | Scope |
|-------------|----------|-------|
| **Browser Cache** | Few minutes (browser-dependent) | Single application |
| **Operating System Cache** | Based on TTL | All applications on computer |
| **Router Cache** (if enabled) | Based on TTL | All devices on network |
| **ISP Recursive Resolver Cache** | Based on TTL | All ISP customers |
| **Intermediate DNS Server Caches** | Based on TTL | Regional |

---

## 11. TTL (Time To Live)

TTL tells caches how long to store a DNS record before checking again.

```
example.com.    300    A    93.184.216.34
                ^^^
                TTL in seconds (5 minutes)
```

### How TTL Works

```
Time 0:00 - Resolver queries authoritative server
          - Gets answer with TTL=300
          - Caches answer
          
Time 0:01 - Another query for same domain
          - Resolver returns cached answer (doesn't query again)
          - TTL now 299 seconds
          
Time 5:00 - TTL expires (reaches 0)
          - Next query will contact authoritative server again
```

### TTL Strategy

| TTL Level | Duration | Pros | Cons | Use Case |
|-----------|----------|------|------|----------|
| **High TTL** | 1 day to 1 week | Less DNS queries, faster response, lower server load | Changes take longer to propagate | Stable infrastructure |
| **Low TTL** | 5-60 minutes | Changes propagate quickly | More DNS queries, higher server load | Before planned changes, dynamic infrastructure |

### Best Practice Before Migration

```
1 week before: Reduce TTL to 5 minutes
During migration: Make DNS changes
After migration: Changes propagate in 5 minutes
1 day after: Increase TTL back to normal
```

### Negative Caching

DNS also caches **negative responses** (NXDOMAIN - domain doesn't exist):

```
Query: nonexistent.example.com
Response: NXDOMAIN (TTL: 300)

This negative answer is cached too!
Next 5 minutes: Queries immediately return NXDOMAIN without checking
```

**Why This Matters**:
- Reduces load from typos and scanning attacks
- Can cause issues: If you create a new subdomain, may not be accessible for TTL duration

---

## 12. Common DNS Commands

### Query DNS records

```bash
# Simple lookup
nslookup example.com

# More detailed (preferred)
dig example.com

# Query specific record type
dig example.com MX
dig example.com TXT

# Query specific DNS server
dig @8.8.8.8 example.com

# Reverse DNS lookup
dig -x 8.8.8.8

# Get all records
dig example.com ANY
```

### Check DNS propagation

```bash
# Query multiple DNS servers
dig @8.8.8.8 example.com        # Google
dig @1.1.1.1 example.com        # Cloudflare
dig @208.67.222.222 example.com # OpenDNS
```

### Flush DNS cache

```bash
# Mac
sudo dscacheutil -flushcache

# Linux
sudo systemd-resolve --flush-caches

# Windows
ipconfig /flushdns
```

---

## 13. Public DNS Servers

| Provider | Primary | Secondary |
|----------|---------|-----------|
| Google | 8.8.8.8 | 8.8.4.4 |
| Cloudflare | 1.1.1.1 | 1.0.0.1 |
| OpenDNS | 208.67.222.222 | 208.67.220.220 |
| Quad9 | 9.9.9.9 | 149.112.112.112 |

---

## 14. DNS in DevOps

### 1. Internal DNS (Private)

In cloud environments:

```
database.internal       → 10.0.2.15
redis.internal          → 10.0.2.20
api.internal            → 10.0.1.10
```

### 2. Service Discovery

Kubernetes uses DNS internally:

```
my-service.default.svc.cluster.local → 10.96.0.10
```

### 3. Blue-Green Deployments

```
# Before deployment
api.example.com → 1.2.3.4 (Blue environment)

# After deployment (update DNS)
api.example.com → 5.6.7.8 (Green environment)
```

### 4. Route53 / Cloud DNS

AWS Route53 example use cases:

| Routing Type | Description |
|--------------|-------------|
| **Weighted routing** | 90% to v1, 10% to v2 |
| **Geolocation routing** | US users to US servers |
| **Failover routing** | Primary/secondary |
| **Latency-based routing** | Route to lowest latency server |

---

## 15. Common DNS Issues

### 1. DNS Propagation Delay

After updating DNS, changes take time (up to 48 hours globally).

**Solution**: Lower TTL before making changes.

### 2. DNS Cache Poisoning

Attackers inject fake DNS records.

**Solution**: Use DNSSEC (DNS Security Extensions).

### 3. Wrong DNS Records

Website not loading after deployment.

**Solution**:

```bash
# Check what DNS returns
dig your-domain.com

# Flush local DNS cache
# Mac:
sudo dscacheutil -flushcache

# Linux:
sudo systemd-resolve --flush-caches

# Windows:
ipconfig /flushdns
```

### 4. DNS Resolution Failures

```bash
# Check if DNS is working
ping your-domain.com

# Trace DNS path
dig +trace your-domain.com

# Check nameservers
dig NS your-domain.com
```

---

## 16. DNS Resolution Example (Step by Step)

**Query: www.shop.example.com**

```
1. Browser cache: Not found
2. OS cache: Not found
3. Router cache: Not found
4. ISP Resolver: Not found
5. Root server: "Go ask .com"
6. .com TLD: "Go ask example.com nameservers"
7. example.com NS: "shop.example.com is at 10.0.1.5"
8. Returns: 10.0.1.5
9. Browser connects to 10.0.1.5
```

---

## 17. Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│                    KEY TAKEAWAYS                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  DNS translates domain names to IP addresses           │
│                                                         │
│  A records point to IPv4, AAAA to IPv6                 │
│                                                         │
│  CNAME creates aliases                                  │
│                                                         │
│  TTL controls caching duration                          │
│                                                         │
│  DNS is critical for service discovery in microservices│
│                                                         │
│  Always test DNS changes before going live             │
│                                                         │
│  Use dig or nslookup for troubleshooting               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Reference Cheat Sheet

| Concept | Details |
|---------|---------|
| **A Record** | IPv4 address mapping |
| **AAAA Record** | IPv6 address mapping |
| **CNAME** | Domain alias |
| **MX Record** | Mail server |
| **TTL** | Cache duration in seconds |
| **Recursive Query** | Client to resolver |
| **Iterative Query** | Resolver to server |
| **Root Servers** | 13 clusters worldwide |
| **TLD Servers** | .com, .org, .net, etc. |
| **Authoritative Server** | Owns the domain records |

---

## Practice Problems

Try these to test your understanding:

1. **What is the difference between A and AAAA records?**
2. **How does TTL affect DNS propagation?**
3. **What is the purpose of a CNAME record?**
4. **How many steps are in the DNS resolution process?**
5. **What is the difference between recursive and iterative queries?**
6. **How do you flush DNS cache on different operating systems?**
7. **What is DNSSEC and why is it important?**
8. **How does round-robin DNS load balancing work?**
