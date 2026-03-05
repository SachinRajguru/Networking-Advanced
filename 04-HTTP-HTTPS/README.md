
# HTTP/HTTPS & Web Protocols

---

## 1. What is HTTP?

**HTTP** (HyperText Transfer Protocol) is the foundation of data communication on the web. It is how your browser talks to web servers.

HTTP is an **application-layer protocol** that defines how messages are formatted and transmitted between clients (usually web browsers) and servers. It was invented by Tim Berners-Lee in 1989 and has evolved significantly since then.

---

## 2. Core Characteristics of HTTP

### 1. Client-Server Model

```
Client (initiates)  ←→  Server (responds)
- Browser               - Web Server
- Mobile App            - API Server
- CLI tool (curl)       - Backend Service
```

### 2. Request-Response Protocol

- Client sends a **request**
- Server processes and sends a **response**
- Each transaction is independent (unless keep-alive is used)

### 3. Stateless Protocol

HTTP itself has no memory of previous requests:

```
Request 1: GET /page1 → Server responds
Request 2: GET /page2 → Server has no memory of Request 1
```

**Why Stateless?**

| Reason | Benefit |
|--------|---------|
| **Simplicity** | Server doesn't need to maintain state |
| **Scalability** | Any server can handle any request |
| **Reliability** | No issues if server restarts |

**Problem**: Web apps need state (shopping carts, login sessions)

**Solutions**:

| Solution | Description |
|----------|-------------|
| **Cookies** | Client stores state |
| **Sessions** | Server stores state, client holds session ID |
| **Tokens** | JWT, OAuth tokens carry state |
| **Local Storage** | Client-side state |

### 4. Text-Based Protocol (HTTP/1.x)

HTTP/1.x messages are human-readable text:

```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
```

**Note**: HTTP/2 and HTTP/3 use binary format for efficiency.

---

## 3. How HTTP Works (Request-Response Cycle)

```
Step 1: DNS Resolution
www.example.com → 93.184.216.34

Step 2: TCP Connection
Client establishes TCP connection to server:443

Step 3: HTTP Request
Client sends HTTP request over TCP connection

Step 4: Server Processing
Server processes request, accesses resources

Step 5: HTTP Response
Server sends response back to client

Step 6: Connection Handling
- HTTP/1.0: Connection closes
- HTTP/1.1+: Connection may stay open (keep-alive)
```

### HTTP vs HTTPS

| Feature | HTTP | HTTPS |
|---------|------|-------|
| **Port** | 80 | 443 |
| **Encryption** | Unencrypted | Encrypted (TLS/SSL) |
| **Speed** | Fast | Slightly slower (encryption overhead) |
| **Security** | Insecure | Secure |
| **URL** | `http://example.com` | `https://example.com` |

**Rule of thumb**: Always use HTTPS for production.

---

## 4. HTTP Request-Response Flow

```
Client                              Server
  |                                   |
  |------- HTTP Request ------------->|
  |  GET /api/users HTTP/1.1          |
  |  Host: api.example.com            |
  |                                   |
  |<------ HTTP Response -------------|
  |  HTTP/1.1 200 OK                  |
  |  Content-Type: application/json   |
  |  { "users": [...] }               |
```

---

## 5. HTTP Methods

| Method | Purpose | Example Use Case | Idempotent |
|--------|---------|------------------|------------|
| **GET** | Retrieve data | Fetch user profile | Yes |
| **POST** | Create new resource | Register new user | No |
| **PUT** | Update entire resource | Update user profile | Yes |
| **PATCH** | Partially update resource | Update user email only | No |
| **DELETE** | Delete resource | Delete user account | Yes |
| **HEAD** | Get headers only | Check if file exists | Yes |
| **OPTIONS** | Get allowed methods | CORS preflight | Yes |

---

## 6. HTTP Status Codes

### 2xx - Success

| Code | Name | Description |
|------|------|-------------|
| 200 | OK | Request succeeded |
| 201 | Created | Resource created successfully |
| 202 | Accepted | Request accepted for processing |
| 204 | No Content | Success, but no content to return |

### 3xx - Redirection

| Code | Name | Description |
|------|------|-------------|
| 301 | Moved Permanently | Resource moved (update your bookmarks) |
| 302 | Found | Temporary redirect |
| 304 | Not Modified | Use cached version |
| 307 | Temporary Redirect | Same as 302, preserves method |

### 4xx - Client Errors

| Code | Name | Description |
|------|------|-------------|
| 400 | Bad Request | Invalid request syntax |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | You don't have permission |
| 404 | Not Found | Resource doesn't exist |
| 405 | Method Not Allowed | HTTP method not supported |
| 409 | Conflict | Resource conflict |
| 429 | Too Many Requests | Rate limit exceeded |

### 5xx - Server Errors

| Code | Name | Description |
|------|------|-------------|
| 500 | Internal Server Error | Server crashed |
| 501 | Not Implemented | Server doesn't support feature |
| 502 | Bad Gateway | Gateway/proxy error |
| 503 | Service Unavailable | Server overloaded or down |
| 504 | Gateway Timeout | Gateway/proxy timeout |

---

## 7. HTTP Request Structure

```
GET /api/users/123 HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer eyJhbGc...
Content-Type: application/json

{
  "name": "John Doe"
}
```

**Parts**:

| Part | Description |
|------|-------------|
| **Request line** | Method, path, version |
| **Headers** | Metadata about request |
| **Body** | Data (for POST/PUT/PATCH) |

---

## 8. HTTP Response Structure

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 156
Cache-Control: max-age=3600
Set-Cookie: sessionId=abc123

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

---

## 9. Important HTTP Headers

### Request Headers

| Header | Purpose | Example |
|--------|---------|---------|
| **Host** | Target server | `api.example.com` |
| **User-Agent** | Client info | `curl/7.64.1` |
| **Accept** | Expected response format | `application/json` |
| **Authorization** | Authentication | `Bearer token123` |
| **Content-Type** | Body format | `application/json` |
| **Cookie** | Session data | `sessionId=abc123` |
| **Content-Length** | Body size | `1234` |

### Response Headers

| Header | Purpose | Example |
|--------|---------|---------|
| **Content-Type** | Response format | `application/json` |
| **Content-Length** | Size in bytes | `1234` |
| **Cache-Control** | Caching rules | `max-age=3600` |
| **Set-Cookie** | Set cookie | `sessionId=xyz789` |
| **Access-Control-Allow-Origin** | CORS policy | `*` |
| **X-RateLimit-Remaining** | Rate limit info | `99` |
| **ETag** | Entity tag | `"abc123"` |

---

## 10. HTTPS & TLS/SSL

### How HTTPS Works

HTTPS = HTTP + TLS (Transport Layer Security)

### The TLS Handshake Process

```
1. Client Hello
Client → Server: "I want HTTPS. I support these cipher suites..."
- TLS version supported
- List of cipher suites (encryption algorithms)
- Random number (for key generation)

2. Server Hello
Server → Client: "Let's use TLS 1.3 with AES-256-GCM"
- Chosen TLS version
- Chosen cipher suite
- Random number
- SSL Certificate (contains public key)

3. Certificate Verification
Client verifies certificate:
- Is it signed by trusted CA?
- Is it for the correct domain?
- Is it expired?
- Has it been revoked?

4. Key Exchange
Client generates pre-master secret:
- Encrypts it with server's public key (from certificate)
- Sends to server
- Only server can decrypt (has private key)

5. Session Keys Generated
Both sides independently generate session keys:
- Using: client random + server random + pre-master secret
- Results in same symmetric encryption key

6. Encrypted Communication Begins
All further communication encrypted with session key
```

---

## 11. SSL Certificates Explained

An SSL certificate is a digital document that:

1. **Proves identity**: This server is really example.com
2. **Contains public key**: Used for initial encryption
3. **Is signed by CA**: Trusted third party vouches for it

### Certificate Contents

```
Subject: example.com
Issuer: Let's Encrypt Authority X3
Valid From: 2024-01-01
Valid Until: 2024-04-01
Public Key: [4096-bit RSA key]
Signature: [CA's digital signature]
```

### Certificate Chain

```
Your Certificate (example.com)
    ↓ Signed by
Intermediate Certificate (Let's Encrypt Authority)
    ↓ Signed by
Root Certificate (ISRG Root X1)
    ↓
Trusted by browsers (pre-installed)
```

### Types of Certificates

| Type | Verification | Cost | Use Case |
|------|--------------|------|----------|
| **Domain Validation (DV)** | Domain ownership only | Free (Let's Encrypt) | Quick to obtain |
| **Organization Validation (OV)** | Organization exists | Moderate cost | Shows organization name |
| **Extended Validation (EV)** | Rigorous verification | Expensive | Banks, financial institutions |

### Wildcard Certificates

```
Certificate for: *.example.com
Covers:
- api.example.com
- www.example.com
- blog.example.com

Does NOT cover:
- example.com (apex domain)
- sub.api.example.com (nested subdomain)
```

### Why Encryption Performance Isn't a Concern Anymore

**Old Myth**: "HTTPS is slow because of encryption overhead"

**Reality** (Modern HTTPS):

| Factor | Benefit |
|--------|---------|
| **TLS 1.3** | Faster handshake (1 RTT vs 2 RTT) |
| **Session Resumption** | Reuse session keys (0 RTT) |
| **Hardware Acceleration** | CPUs have AES instructions |
| **HTTP/2** | Multiplexing compensates for any overhead |
| **CDNs** | Handle TLS termination at edge |

**Actual Performance Impact**: < 1% with modern infrastructure

---

## 12. HTTP Versions

### HTTP/1.0 (1996)

- **One request per connection**
- Connection closes after each response
- Very inefficient

```
Request 1: Open connection → GET /page.html → Close
Request 2: Open connection → GET /style.css → Close
Request 3: Open connection → GET /script.js → Close

Problem: Opening TCP connections is expensive!
```

### HTTP/1.1 (1997) - Most Common Until Recently

**Improvements over 1.0**:

| Feature | Benefit |
|---------|---------|
| **Persistent Connections (Keep-Alive)** | Reuse TCP connection, avoid handshake overhead |
| **Pipelining** | Send multiple requests without waiting |
| **Chunked Transfer Encoding** | Server can send response in chunks |
| **Virtual Hosting** | One IP address, multiple websites |

**HTTP/1.1 Limitations**:

- Head-of-line blocking
- No request prioritization
- Plain text headers (overhead)
- Only client can initiate requests

### HTTP/2 (2015) - Modern Standard

**Major Improvements**:

| Feature | Description |
|---------|-------------|
| **Multiplexing** | Single TCP connection, multiple requests simultaneously |
| **Binary Protocol** | Efficient, but not human-readable |
| **Header Compression (HPACK)** | Reduces header overhead |
| **Server Push** | Server can push resources proactively |
| **Stream Prioritization** | Client tells server priority of resources |

**HTTP/2 Performance Gains**:

- 30-50% faster page loads
- Better on high-latency connections
- Reduces need for domain sharding

### HTTP/3 (2022) - Latest Standard

**Key Change**: Uses QUIC (UDP) Instead of TCP

**Why Move Away from TCP?**

| TCP Problem | HTTP/3 Solution |
|-------------|-----------------|
| Head-of-Line Blocking | Per-stream recovery |
| Slow Handshake | 0-RTT connection |
| Connection Migration | Switch networks without reconnection |
| No Built-in Encryption | QUIC requires TLS 1.3 |

**HTTP/3 Benefits**:

1. **0-RTT Connection**: Even faster than HTTP/2
2. **Better Loss Recovery**: Per-stream, not connection-wide
3. **Connection Migration**: Switch networks (WiFi to 4G) without reconnection
4. **Built-in Encryption**: QUIC requires TLS 1.3

**HTTP/3 Adoption**: Growing, supported by major CDNs (Cloudflare, Google, Facebook)

---

## 13. Real-World DevOps Examples

### Example 1: API Gateway

```
User → HTTPS:443 → API Gateway → HTTP:8080 → Backend Services
```

External traffic uses HTTPS, internal can use HTTP.

### Example 2: Health Check Endpoint

```bash
# Kubernetes liveness probe
GET /health HTTP/1.1
Host: app-service:8080

Response: 200 OK
```

### Example 3: Load Balancer Setup

```
Client → HTTPS:443 → Load Balancer → HTTP:8080 → App Servers
                    (SSL Termination)
```

### Example 4: Microservices Communication

```
Frontend:8080 → HTTP → Backend:8080 → HTTP → Database:3306
```

---

## 14. Common Commands

### Make HTTP requests

```bash
# Simple GET request
curl https://api.example.com/users

# POST with JSON data
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}'

# See full request/response (debugging)
curl -v https://api.example.com

# Follow redirects
curl -L https://example.com

# Check response time
curl -w "@-" -o /dev/null -s https://example.com <<EOF
    time_total:  %{time_total}s
EOF

# Check headers only
curl -I https://api.example.com

# With custom headers
curl -H "Authorization: Bearer token123" https://api.example.com
```

### Test SSL certificate

```bash
# Check certificate details
openssl s_client -connect example.com:443

# Check certificate expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Check certificate chain
openssl s_client -connect example.com:443 -showcerts
```

### Check HTTP response

```bash
# Get status code only
curl -o /dev/null -s -w "%{http_code}" https://api.example.com

# Get response time
curl -o /dev/null -s -w "%{time_total}" https://api.example.com

# Get all timing info
curl -w "@curl-format.txt" -o /dev/null -s https://api.example.com
```

---

## 15. RESTful API Conventions

REST uses HTTP methods semantically:

```
GET    /api/users           # List all users
GET    /api/users/123       # Get user 123
POST   /api/users           # Create new user
PUT    /api/users/123       # Update user 123 (full)
PATCH  /api/users/123       # Update user 123 (partial)
DELETE /api/users/123       # Delete user 123
```

### REST Principles

| Principle | Description |
|-----------|-------------|
| **Stateless** | Each request contains all needed information |
| **Client-Server** | Separation of concerns |
| **Cacheable** | Responses can be cached |
| **Uniform Interface** | Consistent API design |
| **Layered System** | Can have multiple layers (load balancer, proxy) |

---

## 16. CORS (Cross-Origin Resource Sharing)

Allows web pages to request resources from different domains.

### How CORS Works

```
# Browser sends preflight request
OPTIONS /api/data HTTP/1.1
Origin: https://frontend.com

# Server responds
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://frontend.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

### Common CORS Headers

| Header | Purpose | Example |
|--------|---------|---------|
| **Access-Control-Allow-Origin** | Which domains can access | `*` or `https://frontend.com` |
| **Access-Control-Allow-Methods** | Allowed HTTP methods | `GET, POST, PUT, DELETE` |
| **Access-Control-Allow-Headers** | Allowed request headers | `Content-Type, Authorization` |
| **Access-Control-Allow-Credentials** | Allow cookies/auth | `true` or `false` |
| **Access-Control-Max-Age** | Cache preflight response | `86400` (24 hours) |

### DevOps Context

- Configure CORS in your API gateway or backend
- Never use `*` in production with credentials
- Test CORS with browser DevTools Network tab

---

## 17. Caching

### Cache-Control Header

| Value | Description |
|-------|-------------|
| **public** | Response can be cached by any cache |
| **private** | Response can only be cached by client |
| **no-cache** | Must revalidate with server before use |
| **no-store** | Never store response |
| **max-age=3600** | Cache for 1 hour |
| **must-revalidate** | Must check with server after expiry |

```
Cache-Control: public, max-age=3600       # Cache for 1 hour
Cache-Control: private, no-cache          # Don't cache
Cache-Control: no-store                   # Never store
Cache-Control: no-cache, must-revalidate  # Revalidate on each request
```

### ETag (Entity Tag)

```
# First request
Response: ETag: "abc123"

# Next request
Request: If-None-Match: "abc123"
Response: 304 Not Modified (use cached version)
```

### Cache-Control Best Practices

| Resource Type | Recommended Cache |
|---------------|-------------------|
| **Static Assets** (CSS, JS, images) | Long cache (1 year) with versioning |
| **HTML Pages** | Short cache or no-cache |
| **API Responses** | Depends on data freshness |
| **User Data** | No-cache or private |
| **Authentication Tokens** | No-store |

---

## 18. WebSockets

Real-time, bidirectional communication over a single TCP connection.

### WebSocket Handshake

```
Client → Server: HTTP Upgrade request
GET /ws HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

Server → Client: 101 Switching Protocols
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

[Connection upgraded to WebSocket]
Client ⟷ Server: Real-time messages
```

### WebSocket vs HTTP

| Feature | HTTP | WebSocket |
|---------|------|-----------|
| **Direction** | Client → Server only | Bidirectional |
| **Connection** | Short-lived | Long-lived |
| **Overhead** | High per request | Low after handshake |
| **Use Case** | Request-response | Real-time updates |

### WebSocket Use Cases

| Use Case | Description |
|----------|-------------|
| **Chat Apps** | Real-time messaging |
| **Live Dashboards** | Real-time metrics |
| **Gaming** | Multiplayer game state |
| **Stock Tickers** | Real-time price updates |
| **Collaboration Tools** | Live document editing |

---

## 19. Common Issues & Debugging

### Issue: 502 Bad Gateway

```bash
# Check if backend is running
curl http://localhost:8080

# Check backend logs
docker logs backend-container

# Check load balancer configuration
kubectl describe svc backend-service
```

### Issue: SSL Certificate Error

```bash
# Bypass SSL verification (testing only!)
curl -k https://example.com

# Check certificate
openssl s_client -connect example.com:443

# Check certificate expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Check certificate chain
openssl s_client -connect example.com:443 -showcerts
```

### Issue: CORS Error

Add headers to your API:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
```

### Issue: Slow API Response

```bash
# Check response time
curl -w "@curl-format.txt" -o /dev/null -s https://api.example.com

# Check DNS resolution time
dig example.com

# Check TCP connection time
curl -v https://api.example.com 2>&1 | grep "time"

# Check server-side logs
tail -f /var/log/nginx/error.log
```

### Issue: Connection Timeout

```bash
# Increase timeout
curl --connect-timeout 30 --max-time 60 https://api.example.com

# Check if server is reachable
telnet api.example.com 443

# Check firewall rules
iptables -L -n
```

### Issue: Rate Limiting

```bash
# Check rate limit headers
curl -I https://api.example.com

# Common rate limit headers
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1609459200
```

---

## 20. Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│                    KEY TAKEAWAYS                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  HTTP is stateless - each request is independent       │
│                                                         │
│  HTTPS encrypts traffic - always use it in production  │
│                                                         │
│  Status codes tell you what happened                   │
│  (2xx=success, 4xx=client error, 5xx=server error)     │
│                                                         │
│  REST APIs use HTTP methods semantically               │
│                                                         │
│  Headers carry metadata about requests/responses       │
│                                                         │
│  Understanding HTTP is crucial for debugging API issues│
│                                                         │
│  Use curl for testing and debugging HTTP endpoints     │
│                                                         │
│  HTTP/2 and HTTP/3 provide significant performance gains│
│                                                         │
│  CORS must be configured for cross-origin requests     │
│                                                         │
│  Caching improves performance but requires careful management│
│                                                         │
│  WebSockets enable real-time bidirectional communication│
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Reference Cheat Sheet

| Concept | Details |
|---------|---------|
| **HTTP** | Application-layer protocol for web communication |
| **HTTPS** | HTTP + TLS/SSL encryption |
| **Port 80** | HTTP |
| **Port 443** | HTTPS |
| **GET** | Retrieve data (idempotent) |
| **POST** | Create resource (not idempotent) |
| **PUT** | Update entire resource (idempotent) |
| **PATCH** | Partial update (not idempotent) |
| **DELETE** | Delete resource (idempotent) |
| **200 OK** | Success |
| **201 Created** | Resource created |
| **301 Moved** | Permanent redirect |
| **304 Not Modified** | Use cached version |
| **400 Bad Request** | Invalid request |
| **401 Unauthorized** | Authentication required |
| **403 Forbidden** | No permission |
| **404 Not Found** | Resource doesn't exist |
| **429 Too Many Requests** | Rate limit exceeded |
| **500 Internal Error** | Server error |
| **502 Bad Gateway** | Proxy error |
| **503 Service Unavailable** | Server down |
| **HTTP/1.1** | Text-based, keep-alive |
| **HTTP/2** | Binary, multiplexing, header compression |
| **HTTP/3** | QUIC/UDP, 0-RTT, better loss recovery |
| **curl** | Test HTTP endpoints |
| **openssl** | Test SSL certificates |
| **ETag** | Cache validation |
| **CORS** | Cross-origin resource sharing |

---

## Practice Problems

Try these to test your understanding:

1. **What is the difference between HTTP and HTTPS?**
2. **Explain the HTTP request-response cycle**
3. **What is the difference between PUT and PATCH?**
4. **How does HTTP/2 improve performance over HTTP/1.1?**
5. **What is the purpose of the TLS handshake?**
6. **How do you debug a 502 Bad Gateway error?**
7. **What is CORS and why is it needed?**
8. **How does caching work with ETags?**
9. **What are the main differences between HTTP/2 and HTTP/3?**
10. **How do you test SSL certificate expiry?**
