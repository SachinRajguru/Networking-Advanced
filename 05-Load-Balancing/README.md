
# Load Balancing

---

## 1. What is Load Balancing?

**Load balancing** distributes incoming traffic across multiple servers to ensure no single server gets overwhelmed.

Load balancing is a fundamental concept in distributed systems that solves several critical problems:

| Problem | Solution |
|---------|----------|
| **Scalability** | One server has limits; multiple servers scale horizontally |
| **Availability** | If one server fails, others handle the traffic |
| **Performance** | Distributing load prevents bottlenecks |
| **Resource Optimization** | Efficiently utilize all available servers |

### The Core Problem

```
Without Load Balancer:
All traffic → Single Server
- Server gets 1000 req/s
- Server can handle 500 req/s
- Result: 500 req/s dropped, slow responses, server crash

With Load Balancer:
Traffic → Load Balancer → Server 1 (333 req/s)
                       → Server 2 (333 req/s)
                       → Server 3 (333 req/s)
- Each server within capacity
- Fast responses
- System remains stable
```

---

## 2. Load Balancing Theory

### Horizontal Scaling vs Vertical Scaling

| Scaling Type | Description | Pros | Cons |
|--------------|-------------|------|------|
| **Vertical Scaling** (Scale Up) | Upgrade one server: More CPU, RAM, Disk | Simple | Limited by hardware maximums, Expensive, Single point of failure, Downtime during upgrades |
| **Horizontal Scaling** (Scale Out) | Add more servers of same size | Nearly unlimited scaling, Cost-effective, No single point of failure, No downtime to add servers | Requires load balancing, More complex architecture |

**Load balancing enables horizontal scaling.**

---

## 3. Load Balancing Flow

```
                    Load Balancer
                         |
        ┌────────────────┼────────────────┐
        |                |                |
    Server 1         Server 2         Server 3
    (Active)         (Active)         (Active)
```

**Flow**:

1. User sends request to load balancer
2. Load balancer picks a server (using an algorithm)
3. Request is forwarded to chosen server
4. Server processes and responds
5. Load balancer returns response to user

---

## 4. Load Balancing Algorithms

Load balancing algorithms determine how traffic is distributed. Each algorithm optimizes for different scenarios.

### 1. Round Robin

Distributes requests equally in sequence, like dealing cards.

**Algorithm**:
```
servers = [Server1, Server2, Server3]
current = 0

for each request:
    send to servers[current]
    current = (current + 1) % number_of_servers
```

**Example**:
```
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1 (cycle repeats)
Request 5 → Server 2
```

| Advantages | Disadvantages |
|------------|---------------|
| Simple to implement | Doesn't consider server load |
| Fair distribution over time | Doesn't consider request complexity |
| No overhead | Treats all requests as equal |

**Use case**: All servers have similar capacity and all requests take similar time.

---

### 2. Least Connections

Sends traffic to server with fewest active connections.

**Algorithm**:
```
for each request:
    find server with minimum active_connections
    send request to that server
    increment active_connections for that server
    
when response completes:
    decrement active_connections
```

**Example**:
```
Server 1: 5 active connections
Server 2: 2 active connections ← Choose this
Server 3: 8 active connections

New request → Server 2 (now has 3 connections)
```

| Advantages | Disadvantages |
|------------|---------------|
| Better than round-robin for varying request times | More complex |
| Adapts to actual server load | Requires tracking connection count |
| Prevents overloading of servers | May not reflect actual CPU/memory usage |

**Use case**: Long-lived connections with varying processing times (database queries, API calls).

---

### 3. Weighted Round Robin

Servers with higher capacity get more requests.

**Algorithm**:
```
servers = [
    {server: Server1, weight: 3},
    {server: Server2, weight: 2},
    {server: Server3, weight: 1}
]

total_weight = 6
Server1 gets 3/6 = 50% of traffic
Server2 gets 2/6 = 33% of traffic
Server3 gets 1/6 = 17% of traffic
```

**Why Weights?**
```
Server 1: 32 GB RAM, 16 CPUs → Weight 3
Server 2: 16 GB RAM, 8 CPUs  → Weight 2
Server 3: 8 GB RAM, 4 CPUs   → Weight 1

Distribute proportionally to capacity
```

**Use case**: Servers with different capacities (mixed hardware, spot instances).

---

### 4. IP Hash

Routes same client IP to same server.

**Algorithm**:
```
hash_value = hash(client_ip)
server_index = hash_value % number_of_servers
send to servers[server_index]
```

**Example**:
```
Client 203.0.113.5:
hash(203.0.113.5) = 12345
12345 % 3 = 0
Always goes to Server 0

Client 198.51.100.10:
hash(198.51.100.10) = 67890
67890 % 3 = 0
Always goes to Server 0 (collision, but consistent)
```

| Advantages | Disadvantages |
|------------|---------------|
| Provides session affinity without cookies | Uneven distribution (hash collisions) |
| Same client always hits same server | Adding/removing servers changes all mappings |
| Stateless (no server-side session tracking) | Many clients behind same NAT hit same server |

**Use case**: Session persistence without sticky sessions, local caching on servers.

---

### 5. Least Response Time

Routes to server with fastest response time and fewest connections.

**Algorithm**:
```
for each request:
    for each server:
        score = active_connections * average_response_time
    send to server with lowest score
```

**Example**:
```
Server 1: 5 connections, 100ms avg → Score: 500
Server 2: 2 connections, 50ms avg  → Score: 100 ← Choose this
Server 3: 3 connections, 200ms avg → Score: 600
```

| Advantages | Disadvantages |
|------------|---------------|
| Most intelligent algorithm | Most complex |
| Adapts to actual server performance | Requires monitoring response times |
| Considers both load and speed | Overhead of calculation |

**Use case**: Heterogeneous environments, performance-critical applications.

---

### 6. Random

Randomly selects a server.

**Algorithm**:
```
server_index = random(0, number_of_servers - 1)
send to servers[server_index]
```

**Surprisingly Effective**:
- With enough requests, distribution becomes even
- Law of large numbers
- No state needed

| Advantages | Disadvantages |
|------------|---------------|
| Simplest algorithm | Short-term unevenness |
| No synchronization needed | No consideration of server state |

**Use case**: Stateless microservices with many small requests.

---

## 5. Types of Load Balancers

Load balancers operate at different layers of the OSI model, with different capabilities.

### Layer 4 (Transport Layer) - TCP/UDP Load Balancing

**What it does**:
- Routes based on IP address and port number
- No understanding of application protocol
- Looks at TCP/UDP headers only

**How it works**:
```
Client connects to: 203.0.113.10:80
Load Balancer sees:
- Source: 198.51.100.5:54321
- Destination: 203.0.113.10:80
- Protocol: TCP

Decisions based only on:
- IP addresses
- Port numbers
- Connection count
- No inspection of HTTP headers or payload
```

| Advantages | Disadvantages |
|------------|---------------|
| Fast: Minimal processing per packet | No content-based routing |
| Protocol agnostic: Works with any TCP/UDP application | No caching |
| Low latency: Simple packet forwarding | Limited health checks |
| High throughput: Can handle millions of connections | No SSL termination |
| Secure: Doesn't need to decrypt HTTPS | |

**Use cases**:
- High-performance requirements
- Non-HTTP protocols (databases, game servers, SSH)
- Simple load distribution
- Pass-through TLS/SSL

**Example**: AWS NLB (Network Load Balancer), HAProxy in TCP mode

---

### Layer 7 (Application Layer) - HTTP/HTTPS Load Balancing

**What it does**:
- Routes based on HTTP content (URLs, headers, cookies)
- Full understanding of HTTP protocol
- Can modify requests/responses

**How it works**:
```
Client → Load Balancer (terminates HTTP connection)

Load Balancer examines:
- HTTP method (GET, POST)
- URL path (/api/users, /api/orders)
- Headers (Host, User-Agent, Cookie)
- Query parameters
- Body content (JSON, XML)

Makes routing decision based on content
Opens new connection to backend
```

**Request Inspection**:
```
GET /api/users/123 HTTP/1.1
Host: api.example.com
Cookie: session=abc123
User-Agent: Mozilla/5.0

Load Balancer can route based on:
- Path starts with /api/users → User Service
- Path starts with /api/orders → Order Service
- Cookie contains session → Sticky session
- User-Agent contains "Mobile" → Mobile backend
- Header contains "Admin" → Admin backend
```

| Advantages | Disadvantages |
|------------|---------------|
| Smart routing: Route by URL, header, cookie, method | Slower: Must parse HTTP, more processing |
| SSL termination: Decrypt once at load balancer, backend uses HTTP | SSL overhead: Must decrypt/re-encrypt (if backend HTTPS) |
| Caching: Cache responses at load balancer | Connection termination: Breaks end-to-end connection |
| Compression: Compress responses | More complex: More configuration options |
| Security: WAF, rate limiting, authentication | Higher cost: Requires more resources |

**Use cases**:
- Microservices (route by path)
- A/B testing (route by cookie/header)
- Geographic routing (route by IP)
- Content-based routing
- SSL offloading

**Example**: AWS ALB (Application Load Balancer), Nginx, Apache

---

### Layer 4 vs Layer 7 Comparison

| Feature | Layer 4 | Layer 7 |
|---------|---------|---------|
| **Performance** | 1,000,000+ connections/second | 100,000+ connections/second |
| **Routing** | IP:Port → Backend pool | URL, headers, cookies → Backend pool |
| **Health Checks** | TCP connection succeeds/fails | HTTP status, response body, response time |
| **SSL** | Pass-through | Termination at load balancer |
| **Caching** | No | Yes |
| **Content Modification** | No | Yes |

**When to Use Each**:

| Use Layer 4 When | Use Layer 7 When |
|------------------|------------------|
| Maximum performance needed | Need content-based routing |
| Simple load distribution sufficient | Multiple services behind one IP |
| Non-HTTP protocols | Want SSL termination |
| Don't need to inspect traffic | Need caching or compression |
| Want minimal latency | Microservices architecture |
| | Need advanced features (WAF, rate limiting) |

---

## 6. Real-World Examples

### Example 1: Basic Web Application

```
Internet
   |
   ↓
Load Balancer (nginx/HAProxy)
   |
   ├→ Web Server 1 (app:8080)
   ├→ Web Server 2 (app:8080)
   └→ Web Server 3 (app:8080)
```

### Example 2: Microservices Architecture

```
API Gateway (Load Balancer)
   |
   ├→ /users/* → User Service (3 instances)
   ├→ /orders/* → Order Service (2 instances)
   └→ /products/* → Product Service (4 instances)
```

### Example 3: Multi-Region Setup

```
Global Load Balancer (Route53, Cloudflare)
   |
   ├→ US East → Regional Load Balancer → Servers
   ├→ EU West → Regional Load Balancer → Servers
   └→ Asia Pacific → Regional Load Balancer → Servers
```

---

## 7. Health Checks

Load balancers regularly check if servers are healthy and able to handle traffic. This is critical for high availability.

### Why Health Checks Matter

**Without Health Checks**:
```
Server 1: ✓ Running
Server 2: ✗ Crashed
Server 3: ✓ Running

Load Balancer still sends traffic to Server 2
→ 33% of requests fail
→ Users see errors
```

**With Health Checks**:
```
Server 1: ✓ Running (Healthy)
Server 2: ✗ Crashed (Unhealthy) ← Removed from pool
Server 3: ✓ Running (Healthy)

Load Balancer only sends to Server 1 and Server 3
→ 0% of requests fail
→ Users don't notice Server 2 is down
```

---

### Types of Health Checks

#### 1. TCP Health Check (Layer 4)

**How it works**:
```
Every X seconds:
  Load Balancer → TCP SYN to Server:Port
  
  If Server responds with SYN-ACK:
    Server is Healthy
  
  If timeout or connection refused:
    Server is Unhealthy
```

**Example**:
```
Check: Connect to 10.0.1.5:8080
Success → Healthy (server is listening)
Timeout → Unhealthy (server down or port closed)
```

| Advantages | Disadvantages |
|------------|---------------|
| Fast and simple | Only checks if port is open |
| Low overhead | Doesn't verify application is working |
| Works for any TCP service | False positives possible |

**Use case**: Basic connectivity checks, non-HTTP services.

---

#### 2. HTTP/HTTPS Health Check (Layer 7)

**How it works**:
```
Every X seconds:
  Load Balancer → GET /health HTTP/1.1
  
  Server responds:
    HTTP/1.1 200 OK
    {"status": "healthy"}
  
  Load Balancer checks:
    - Status code (is it 200?)
    - Response time (< timeout?)
    - Response body (matches expected?)
  
  If all pass:
    Server is Healthy
  Else:
    Server is Unhealthy
```

**Example Health Check Endpoint**:
```json
GET /health

Response (Healthy):
HTTP/1.1 200 OK
{
  "status": "healthy",
  "database": "connected",
  "cache": "connected",
  "disk_space": "sufficient"
}

Response (Unhealthy):
HTTP/1.1 503 Service Unavailable
{
  "status": "unhealthy",
  "database": "disconnected",
  "cache": "connected"
}
```

| Advantages | Disadvantages |
|------------|---------------|
| Application-level check | Higher overhead |
| Can verify dependencies (database, cache, etc.) | More complex to implement |
| Can check application logic | Slower than TCP check |
| Detailed status information | |

**Use case**: Web applications, APIs, microservices.

---

### Health Check Parameters

| Parameter | Description | Short | Long |
|-----------|-------------|-------|------|
| **Interval** | Time between checks | Faster detection | Less overhead |
| **Timeout** | Maximum wait time for response | Quick failure detection | Tolerates slow responses |
| **Unhealthy Threshold** | Consecutive failures before marking unhealthy | Prevents false positives | Quick removal |
| **Healthy Threshold** | Consecutive successes before marking healthy | Prevents flapping | Faster recovery |

**Best Practice**:
```
Interval: 10 seconds
Timeout: 5 seconds
Unhealthy Threshold: 3
Healthy Threshold: 2

Takes 30 seconds to mark unhealthy
Takes 20 seconds to mark healthy
```

---

### Health Check State Machine

```
         Initial State
              |
              ↓
         [Healthy] ←──────────────┐
              |                   |
    Fail (count = 1)         Pass (count = 1)
              ↓                   |
    Fail (count = 2)         Pass (count = 2)
              ↓                   |
    Fail (count = 3)         [Healthy Threshold Met]
              ↓                   
        [Unhealthy] ────────────→
```

---

### Advanced Health Checks

#### 1. Active vs Passive Health Checks

| Type | Description |
|------|-------------|
| **Active** | Load balancer proactively sends health check requests at regular intervals (e.g., every 10 seconds). Predictable overhead. |
| **Passive** | Load balancer monitors actual traffic. If server returns errors → Mark unhealthy. No extra requests. Only works when traffic exists. |

#### 2. Graceful Degradation

```
Health endpoint can return different statuses:
- 200: Fully healthy (accept all traffic)
- 429: Degraded (accept reduced traffic)
- 503: Unhealthy (remove from pool)

Advanced load balancers respect this
```

#### 3. Dependency Checks

Health endpoint checks critical dependencies:

```python
def health_check():
    checks = {
        "database": check_database(),
        "cache": check_cache(),
        "external_api": check_external_api()
    }
    
    if all(checks.values()):
        return 200, "healthy"
    elif checks["database"]:  # Database is critical
        return 503, "unhealthy"
    else:  # Non-critical service down
        return 429, "degraded"
```

---

### Best Practices

| Practice | Description |
|----------|-------------|
| **Separate Health Check Endpoint** | Use `/health` or `/healthz`, not homepage |
| **Lightweight Health Checks** | Quick database query (SELECT 1), not full scans |
| **Include Warmup Time** | Give server time to warm up caches before marking healthy |
| **Monitor Health Check Metrics** | Track success rate, response times, flapping |
| **Different Checks for Different Needs** | Liveness, Readiness, Startup checks |

**Liveness vs Readiness vs Startup**:

| Check Type | Purpose | Action on Failure |
|------------|---------|-------------------|
| **Liveness** | Is the process running? | Restart the container |
| **Readiness** | Is it ready for traffic? | Remove from load balancer pool |
| **Startup** | Has it finished initializing? | Wait before checking |

---

## 8. Session Persistence (Sticky Sessions)

Keep user connected to same server for entire session.

### Cookie-Based Sticky Sessions

```
First Request:
Client → Load Balancer → Server 2
Response: Set-Cookie: SERVER_ID=server2

Subsequent Requests:
Client (Cookie: SERVER_ID=server2) → Load Balancer → Server 2
```

| Pros | Cons |
|------|------|
| User session data stays on one server | Uneven load distribution |
| Simple to implement | No failover if server crashes |
| No server-side session tracking | Can cause hotspots |

**Alternative: Application-Level Session Storage**

```
Store session in Redis/Memcached
All servers can access session data
No sticky sessions needed
```

---

## 9. Popular Load Balancers

### Software Load Balancers

| Load Balancer | Type | Best For |
|---------------|------|----------|
| **Nginx** | Layer 7 | Web servers, reverse proxy, load balancer |
| **HAProxy** | Layer 4/7 | High-performance TCP/HTTP load balancing |
| **Traefik** | Layer 7 | Cloud-native, automatic service discovery |
| **Envoy** | Layer 7 | Service mesh, microservices |
| **Apache** | Layer 7 | Web server with mod_proxy |

### Cloud Load Balancers

| Provider | Layer 4 | Layer 7 |
|----------|---------|---------|
| **AWS** | NLB (Network Load Balancer) | ALB (Application Load Balancer) |
| **Azure** | Load Balancer | Application Gateway |
| **GCP** | Cloud Load Balancing | Cloud Load Balancing |

### Kubernetes Load Balancing

| Component | Purpose |
|-----------|---------|
| **Service** | Internal load balancing |
| **Ingress** | External HTTP/HTTPS routing |
| **Service Mesh** | Advanced traffic management (Istio, Linkerd) |

---

## 10. Simple Nginx Load Balancer Config

```nginx
upstream backend {
    # Load balancing algorithm (default: round-robin)
    # least_conn;
    # ip_hash;
    
    server 10.0.1.10:8080;
    server 10.0.1.11:8080;
    server 10.0.1.12:8080;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Health check
        proxy_next_upstream error timeout http_502 http_503 http_504;
    }
}
```

---

## 11. Load Balancing Patterns

### 1. Active-Active

All servers handle traffic simultaneously.

```
Load Balancer → [Server 1, Server 2, Server 3] (all active)
```

### 2. Active-Passive

One server handles traffic, others are standby.

```
Load Balancer → Server 1 (active)
                Server 2 (standby)
                Server 3 (standby)
```

### 3. Geographic Load Balancing

Route users to nearest region.

```
User in USA → US Load Balancer
User in Europe → EU Load Balancer
User in Asia → Asia Load Balancer
```

### 4. Blue-Green Deployment

```
Blue Environment (Current) → Traffic
Green Environment (New) → No traffic

After testing:
Blue Environment → No traffic
Green Environment (New) → Traffic
```

### 5. Canary Deployment

```
90% → Blue Environment (Current)
10% → Green Environment (New)

Gradually increase Green percentage
```

---

## 12. Auto-Scaling with Load Balancers

```
1. Load increases → Metrics trigger scaling
2. New servers are launched
3. Load balancer adds them to pool (after health check)
4. Traffic distributed to all servers
5. Load decreases → Servers removed
```

**Auto-Scaling Flow**:

```
┌─────────────────────────────────────────────────────────┐
│                    Auto-Scaling Flow                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. CPU/Memory exceeds threshold (e.g., 80%)           │
│  2. Auto-scaling group triggers                        │
│  3. New server instance launched                       │
│  4. Server passes health checks                        │
│  5. Load balancer adds server to pool                  │
│  6. Traffic distributed to all servers                 │
│  7. Load decreases → Scale down                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 13. Common Commands & Tools

### Test load balancing

```bash
# Send multiple requests and see which server responds
for i in {1..10}; do
  curl -s http://loadbalancer.example.com | grep "Server ID"
done

# Test with different IPs (if using IP hash)
curl --interface eth0 http://loadbalancer.example.com
curl --interface eth1 http://loadbalancer.example.com

# Check response time
curl -w "@curl-format.txt" -o /dev/null -s http://loadbalancer.example.com
```

### Check load balancer status

```bash
# HAProxy stats page
curl http://loadbalancer:9000/stats

# Check backend servers
curl -I http://backend-server:8080/health

# View active connections
ss -tuln | grep :80
```

### Test health checks

```bash
# Test health endpoint
curl -I http://backend-server:8080/health

# Test with timeout
curl --connect-timeout 5 http://backend-server:8080/health

# Check response time
curl -w "Time: %{time_total}s\n" -o /dev/null -s http://backend-server:8080/health
```

---

## 14. Troubleshooting

### Issue: Uneven Load Distribution

**Possible causes**:
- Sticky sessions enabled
- Long-lived connections
- IP hash with few clients

**Solution**: Change algorithm or disable sticky sessions.

---

### Issue: Server Marked Unhealthy

```bash
# Check if server is actually running
curl http://backend-server:8080/health

# Check health check configuration
# Ensure timeout and interval are appropriate

# Check server logs
tail -f /var/log/app.log

# Check resource usage
top
htop
```

---

### Issue: 502 Bad Gateway

**Possible causes**:
- All backend servers are down
- Backend servers timing out
- Network issue between LB and servers

**Solution**: Check backend logs and connectivity.

```bash
# Check backend connectivity
curl http://backend-server:8080

# Check network
ping backend-server
traceroute backend-server

# Check firewall
iptables -L -n
```

---

### Issue: SSL Certificate Errors

```bash
# Check certificate
openssl s_client -connect loadbalancer:443

# Check certificate expiry
echo | openssl s_client -connect loadbalancer:443 2>/dev/null | openssl x509 -noout -dates

# Test SSL configuration
openssl s_client -connect loadbalancer:443 -tls1_2
```

---

### Issue: High Latency

```bash
# Check response times
curl -w "@curl-format.txt" -o /dev/null -s http://loadbalancer.example.com

# Check DNS resolution
dig loadbalancer.example.com

# Check TCP connection time
curl -v http://loadbalancer.example.com 2>&1 | grep "time"

# Check server-side performance
top
htop
```

---

## 15. Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│                    KEY TAKEAWAYS                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Load balancers distribute traffic across servers      │
│                                                         │
│  Round-robin is simple, least-connections adapts to load│
│                                                         │
│  Layer 4 is fast, Layer 7 offers more features         │
│                                                         │
│  Health checks ensure traffic only goes to healthy servers│
│                                                         │
│  Session persistence can cause uneven distribution     │
│                                                         │
│  Auto-scaling works hand-in-hand with load balancing   │
│                                                         │
│  Always have multiple backend servers for high availability│
│                                                         │
│  Choose algorithm based on your use case               │
│                                                         │
│  Layer 4 for performance, Layer 7 for features         │
│                                                         │
│  Health checks should be lightweight and fast          │
│                                                         │
│  Monitor load balancer metrics continuously            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Reference Cheat Sheet

| Concept | Details |
|---------|---------|
| **Round Robin** | Equal distribution, simple |
| **Least Connections** | Adapts to actual load |
| **Weighted Round Robin** | Proportional to capacity |
| **IP Hash** | Session affinity without cookies |
| **Least Response Time** | Most intelligent, most complex |
| **Layer 4** | Fast, TCP/UDP only |
| **Layer 7** | Smart routing, HTTP/HTTPS |
| **Health Check** | TCP or HTTP endpoint |
| **Sticky Sessions** | Cookie-based persistence |
| **Auto-Scaling** | Scale with load |
| **Active-Active** | All servers handle traffic |
| **Active-Passive** | Standby servers |

---

## Practice Problems

Try these to test your understanding:

1. **What is the difference between Layer 4 and Layer 7 load balancing?**
2. **When would you use least-connections over round-robin?**
3. **How do health checks work and why are they important?**
4. **What are the pros and cons of sticky sessions?**
5. **How does auto-scaling work with load balancers?**
6. **What is the difference between active-active and active-passive?**
7. **How do you troubleshoot a 502 Bad Gateway error?**
8. **What are the best practices for health check endpoints?**
9. **When would you use weighted round-robin?**
10. **How does IP hash provide session affinity?**
