# Kubernetes Networking

---

## 1. What is Kubernetes Networking?

**Kubernetes networking** connects Pods, Services, and external clients in a cluster.

Kubernetes networking is fundamentally different from traditional networking. It's built on a set of requirements that enable seamless communication in a dynamic, container-orchestrated environment.

### The Kubernetes Network Model

Kubernetes imposes these fundamental requirements on any network implementation:

| Requirement | Description |
|-------------|-------------|
| **Every Pod gets its own IP address** | No port mapping needed, unique IP in cluster |
| **Pods can communicate with all other Pods without NAT** | Direct communication, no address translation |
| **Agents on a node can communicate with all Pods** | kubelet, kube-proxy can reach any Pod |
| **Pods in host network see same network as node** | Shares node's network namespace |

### Why This Model?

**Problem with Traditional Port Mapping**:
```
Docker style:
Container A: localhost:8080 → Host port 30001
Container B: localhost:8080 → Host port 30002

Challenges:
- Port allocation management
- Service discovery complex
- No direct container-to-container
- Doesn't scale
```

**Kubernetes Model Benefits**:
```
Each Pod has IP:
Pod A: 10.244.1.5:8080 (actual Pod IP)
Pod B: 10.244.1.6:8080 (actual Pod IP)

Benefits:
- Simple addressing (IP:Port)
- Direct communication
- Scales to thousands of Pods
- Service discovery easier
```

### How Kubernetes Achieves This

**CNI (Container Network Interface)**:
```
Kubernetes doesn't implement networking itself
Delegates to CNI plugins

When Pod starts:
1. Kubernetes calls CNI plugin
2. CNI plugin:
   - Assigns IP to Pod
   - Sets up network interface
   - Configures routing
3. Pod ready with network
```

**Network responsibilities split**:
| Component | Responsibility |
|-----------|----------------|
| **Kubernetes Core** | Defines network model requirements, Manages Service abstraction, Provides DNS |
| **CNI Plugin** | Implements Pod networking, IP address management (IPAM), Cross-node routing, Network policies |

---

## 2. Pods and IPs

### Pod Networking

Every Pod gets its own unique IP address.

**Pod Network Architecture**:
```
Kubernetes Cluster
    |
    ├── Node 1 (10.0.1.10)
    |    └── Pod Subnet: 10.244.1.0/24
    |         ├── Pod A: 10.244.1.5
    |         └── Pod B: 10.244.1.6
    |
    └── Node 2 (10.0.1.20)
         └── Pod Subnet: 10.244.2.0/24
              ├── Pod C: 10.244.2.8
              └── Pod D: 10.244.2.9

Each node gets a Pod subnet
Pods get IPs from node's subnet
Cluster-wide unique IPs
```

**IP Allocation**:
```
Cluster Pod CIDR: 10.244.0.0/16 (all Pods)
    ↓
Per-Node Pod CIDR:
Node 1: 10.244.1.0/24 (254 Pods max)
Node 2: 10.244.2.0/24 (254 Pods max)
Node 3: 10.244.3.0/24 (254 Pods max)

CNI plugin manages allocation
Prevents IP conflicts
```

**Key Point**: Pods are ephemeral - IPs change when Pods restart.

```
Pod "web-app-abc123" → IP: 10.244.1.5
Pod crashes
New Pod "web-app-def456" → IP: 10.244.1.8 (different!)

Problem: How do other Pods find it?
Solution: Services (stable IPs)
```

---

### Containers in a Pod

Containers in the same Pod share the same network namespace.

**Why Pods Share Network Namespace?**

**Problem**: Multiple containers need tight coupling
```
Example: Web server + Log collector
- Web server generates logs
- Log collector reads logs
- Need fast, efficient communication

If separate network namespaces:
- Complex networking setup
- Higher latency
- More overhead
```

**Solution**: Shared network namespace
```
Pod: web-app (IP: 10.244.1.5)
  ├── nginx container
  |    └── Listens on: localhost:80
  └── logger container
       └── Connects to: localhost:80

Both containers:
- Share same IP: 10.244.1.5
- Share same localhost
- Share same network interfaces
- Share same port space
```

**Communication Patterns**:
| Pattern | Description |
|---------|-------------|
| **Container-to-Container (same Pod)** | Uses loopback interface, no network overhead |
| **Pod-to-Pod (different Pods)** | Uses Pod network, goes through CNI |
| **External-to-Pod** | Goes through Service, load balanced |

**Port Conflict Consideration**:
```
Same Pod:
Container 1: Listens on port 80 ✓
Container 2: Listens on port 80 ✗ (conflict!)

Must coordinate ports:
Container 1: Port 80
Container 2: Port 8080
Container 3: Port 9090

Different Pods:
Pod A - Container: Port 80 ✓
Pod B - Container: Port 80 ✓ (no conflict, different network namespace)
```

---

## 3. Services

Services provide stable IP and DNS name for accessing Pods.

### The Problem Services Solve

**Pod IP Instability**:
```
Time 0: Deploy 3 replicas of web-app
  - Pod 1: 10.244.1.5
  - Pod 2: 10.244.1.6
  - Pod 3: 10.244.2.8

Time 1: Pod 1 crashes, replaced
  - Pod 1 (new): 10.244.1.9 ← Different IP!
  - Pod 2: 10.244.1.6
  - Pod 3: 10.244.2.8

Time 2: Scale up to 5 replicas
  - Pod 1: 10.244.1.9
  - Pod 2: 10.244.1.6
  - Pod 3: 10.244.2.8
  - Pod 4: 10.244.1.10 ← New
  - Pod 5: 10.244.2.9  ← New

Problem: 
- How do clients track changing IPs?
- How to load balance across Pods?
- How to handle Pods coming/going?
```

**Service Solution**:
```
Create Service: web-app-service
  - Stable IP: 10.96.0.10 (ClusterIP)
  - Stable DNS: web-app-service.default.svc.cluster.local
  - Automatically tracks backend Pods
  - Load balances requests

Clients connect to: 10.96.0.10 (never changes)
Service routes to: Current healthy Pods
```

### How Services Work (Technical Details)

**Service Creation**:
```
1. Create Service object
2. Kubernetes assigns ClusterIP (from Service CIDR)
3. kube-proxy on each node watches for Service
4. kube-proxy configures iptables/IPVS rules
5. Traffic to ClusterIP → Routed to Pod IPs
```

**Service Components**:

| Component | Description |
|-----------|-------------|
| **ClusterIP** | Virtual IP for the Service (doesn't exist on any interface) |
| **Endpoints** | List of Pod IPs backing the Service (auto-updates) |
| **kube-proxy** | Programs network rules on each node |

**kube-proxy Modes**:

| Mode | Description | Performance |
|------|-------------|-------------|
| **iptables** (default) | Creates iptables rules per Service | Good |
| **IPVS** (more efficient) | Uses Linux IPVS (hash tables) | Better for 1000+ Services |

**Traffic Flow Through Service**:
```
1. Client Pod sends to: 10.96.0.10:80 (Service ClusterIP)

2. Packet hits iptables/IPVS rules on node

3. Rule translates (DNAT):
   Dest: 10.96.0.10:80 → 10.244.1.5:80 (chosen Pod)

4. Packet routed to Pod (CNI networking)

5. Pod processes and responds to: Client IP

6. Response returns through node

7. Node translates back (reverse DNAT):
   Source: 10.244.1.5:80 → 10.96.0.10:80

8. Client receives response from "Service"

From client perspective: Talked to Service (10.96.0.10)
Reality: Talked to Pod (10.244.1.5)
```

---

### Service Types

Services have different types for different access patterns.

#### 1. ClusterIP (Default)

**Characteristics**:
- Internal access only (within cluster)
- Stable virtual IP
- DNS name provided
- Most common type

**Access Pattern**:
```
Any Pod in cluster → Service ClusterIP → Backend Pods

Frontend Pod → backend-service:8080 → Backend Pods
```

**Use Cases**:
- Internal microservices communication
- Databases accessible only from cluster
- Backend APIs not exposed externally

#### 2. NodePort

Exposes service on each Node's IP at a static port.

**How It Works**:
```
Creates ClusterIP + Opens port on every node

Every node listens on NodePort:
Node1 (10.0.1.10):30080 ─┐
Node2 (10.0.1.20):30080 ─┼→ Service → Pods
Node3 (10.0.1.30):30080 ─┘

Traffic to any node:30080 → Routed to Service → Pods
```

**Port Range**: 30000-32767 (by default, configurable)

**externalTrafficPolicy**:

| Policy | Description |
|--------|-------------|
| **Cluster** (default) | Traffic to any node → Can route to Pods on other nodes, Source IP is lost (SNAT) |
| **Local** | Traffic to node → Only routes to Pods on SAME node, Preserves source IP (no SNAT) |

**Use Cases**:
- Development/testing (quick external access)
- Small deployments
- On-premise without load balancer
- Direct node access acceptable

**Limitations**:
- One service per port (30000-32767 range limited)
- Must know node IPs
- No high availability (if node dies, that path fails)
- No SSL termination
- No hostname-based routing

#### 3. LoadBalancer

Creates an external load balancer (cloud provider).

**How It Works**:
```
Cloud Provider Integration:
1. Kubernetes creates LoadBalancer Service
2. Cloud controller provisions external LB
3. LB gets public IP
4. LB routes to NodePorts on cluster nodes
5. Nodes route to Pods

External LB (public IP: 203.0.113.10)
    ↓
Node1:31234 ─┐
Node2:31234 ─┼→ Service → Pods
Node3:31234 ─┘
```

**Two Levels of Load Balancing**:
```
Level 1: External LB
- Balances across cluster nodes
- Health checks nodes
- Removes unhealthy nodes

Level 2: kube-proxy
- Balances across Pods
- Tracks Pod health
- Routes to healthy Pods

Result: Highly available, distributed load balancing
```

**Use Cases**:
- Production web applications
- Public-facing services
- When you need external load balancer
- Cloud environments

**Cost Consideration**:
```
Each LoadBalancer Service = One cloud LB = $$

5 Services × $20/month = $100/month

Alternative: Use Ingress
- One LoadBalancer for all services
- Ingress routes by hostname/path
- Cost effective
```

#### 4. ExternalName

Maps service to DNS name (no proxying).

**How It Works**:
```
Not a typical Service - no ClusterIP, no Pods

Service: external-db
Type: ExternalName
ExternalName: db.external.com

When Pod queries: external-db
DNS returns: CNAME to db.external.com
Pod connects directly to external host
```

**Use Case**:
```
Scenario: Migrating from external DB to in-cluster

Phase 1 (External):
App → external-db → db.external.com (outside cluster)

Phase 2 (Migrated):
Change ExternalName to ClusterIP type
App code unchanged (still uses external-db)
App → external-db → Pods in cluster

Abstraction layer for gradual migration
```

---

## 4. Service Discovery (DNS)

Kubernetes has built-in DNS for service discovery.

### DNS Resolution:

```
Service: backend in namespace: default

Full name: backend.default.svc.cluster.local

Can be accessed as:
- backend (same namespace)
- backend.default (specify namespace)
- backend.default.svc.cluster.local (FQDN)
```

**Example**:
```yaml
# Frontend Pod connecting to backend Service
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: frontend-app
    env:
    - name: BACKEND_URL
      value: "http://backend:8080"  # Service name as hostname
```

---

## 5. Ingress

Ingress exposes HTTP/HTTPS routes to services.

**Think of it as**: Layer 7 load balancer and reverse proxy.

### Without Ingress:
```
Service 1: LoadBalancer (External IP 1)
Service 2: LoadBalancer (External IP 2)
Service 3: LoadBalancer (External IP 3)

Problem: Many load balancers = expensive!
```

### With Ingress:
```
                    Ingress
                       |
    ┌──────────────────┼──────────────────┐
    |                  |                  |
Service 1          Service 2          Service 3
(ClusterIP)        (ClusterIP)        (ClusterIP)

One external IP, routes by URL!
```

### Ingress Example:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 8080
```

**Result**:
- `http://example.com/` → frontend service
- `http://example.com/api` → backend service

### Popular Ingress Controllers:
- **Nginx Ingress** (most popular)
- **Traefik** (cloud-native)
- **HAProxy Ingress**
- **AWS ALB Ingress** (AWS only)
- **Istio Gateway** (service mesh)

### Real-World Examples

#### Example 1: Simple Web Application

This example demonstrates a basic two-tier application with a backend API and a frontend web server.

```yaml
# Backend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: api
        image: my-api:v1
        ports:
        - containerPort: 8080

---
# Backend Service (internal)
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 8080

---
# Frontend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: web
        image: my-frontend:v1
        ports:
        - containerPort: 80
        env:
        - name: API_URL
          value: "http://backend:8080"  # Service DNS

---
# Frontend Service (external)
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
```

#### Example 2: Microservices with Ingress

This example demonstrates routing traffic to different microservices based on URL paths using an Ingress resource.

```yaml
# Ingress routing
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microservices-ingress
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 8080
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 8080
      - path: /products
        pathType: Prefix
        backend:
          service:
            name: product-service
            port:
              number: 8080
```

**Result**:
- `api.example.com/users/*` → user-service
- `api.example.com/orders/*` → order-service
- `api.example.com/products/*` → product-service

---

## 6. Network Policies

Control traffic flow between Pods (like a firewall).

### Default Behavior:
All Pods can communicate with all Pods (no restrictions).

### With Network Policy:
```yaml
# Restrict backend to only accept traffic from frontend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

**Result**: Only Pods with label `app: frontend` can reach backend on port 8080.

### Example: Database Isolation
```yaml
# Database only accessible from backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 5432
```

---

## 7. CNI (Container Network Interface)

CNI plugins provide networking for Kubernetes. Kubernetes doesn't implement networking - it delegates to CNI plugins.

### CNI Architecture

**How CNI Works**:
```
1. Pod is scheduled to Node
2. kubelet creates Pod container(s)
3. kubelet calls CNI plugin via JSON-RPC
4. CNI plugin:
   a. Creates network namespace
   b. Assigns IP from node's CIDR
   c. Sets up veth pair
   d. Configures routes
   e. Sets up network policies (if supported)
5. Returns network config to kubelet
6. Pod is ready with networking
```

**CNI Plugin Interface**:
```
Input (from kubelet):
- Container ID
- Network namespace path
- Network configuration

Output (from plugin):
- IP address assigned
- Gateway
- Routes
- DNS servers

Standard interface - any CNI plugin can work
```

### Popular CNI Plugins

#### Calico

**Architecture**:
```
Layer 3 routing (no overlay):
- Each node is a virtual router (BGP)
- Nodes learn routes to Pods via BGP
- Direct IP routing (no encapsulation)

Node1: 10.244.1.0/24 → Advertises via BGP
Node2: 10.244.2.0/24 → Advertises via BGP

Nodes build routing table:
10.244.1.0/24 via Node1
10.244.2.0/24 via Node2

Result: Efficient, no overlay overhead
```

**BGP Routing**:
```
Border Gateway Protocol (internet routing protocol):

Each node runs BGP daemon (BIRD)
Nodes peer with each other (or route reflectors)
Exchange routes to Pod CIDRs

Benefit: Standard internet routing
Scales to large clusters
```

**Network Policies**:
```
Calico implements NetworkPolicy API:
- iptables rules (default)
- Or eBPF (faster, modern)

Strong network policy support
Micro-segmentation possible
```

**Use Cases**:
- Large clusters (scales well)
- Network policy requirements
- No overlay overhead needed
- BGP routing acceptable

#### Flannel

**Architecture**:
```
Simple overlay networking:
- VXLAN encapsulation (default)
- Each node gets subnet
- Cross-node traffic tunneled

Pod on Node1 → VXLAN tunnel → Pod on Node2
```

**How It Works**:
```
1. Flannel runs on each node (DaemonSet)
2. Flannel allocates subnet to node (from etcd)
3. Creates flannel.1 interface (VXLAN device)
4. Wraps Pod traffic in VXLAN
5. Sends over node network
6. Destination unwraps and delivers

Transparent to Pods
```

**Advantages**:
- Simple setup
- Works everywhere (no special network requirements)
- Stable and mature

**Disadvantages**:
- Overlay overhead (encapsulation)
- Limited network policy support
- Less performant than Calico

**Use Cases**:
- Simple clusters
- Quick setup needed
- No special network requirements
- Network policies not critical

#### Cilium

**Architecture**:
```
eBPF-based networking:
- Programs loaded into Linux kernel
- Packet processing at kernel level
- No iptables (faster)

Traditional:
Packet → iptables rules → Slow

Cilium:
Packet → eBPF program → Very fast
```

**eBPF Benefits**:
```
In-kernel processing:
- No context switches
- No userspace overhead
- Programmable datapath

10-100x faster than iptables
Especially for many rules
```

**Features**:
```
- Layer 3/4 networking
- Layer 7 protocol visibility (HTTP, gRPC, Kafka)
- Network policies (L3, L4, L7)
- Service mesh capabilities
- Observability built-in
```

**Use Cases**:
- High performance requirements
- Layer 7 network policies needed
- Service mesh (Cilium replaces sidecar proxies)
- Advanced observability

#### Weave

**Architecture**:
```
Mesh networking:
- Each node connects to every other node
- Gossip protocol for peer discovery
- Encrypted overlay option

Full mesh = Maximum redundancy
```

**Features**:
```
- Automatic encryption (weave-net)
- Multicast support
- Simple setup
- No external database needed
```

**Use Cases**:
- Multi-cloud (connects across environments)
- Encryption required
- Mesh topology preferred

#### Canal

**Architecture**:
```
Combination: Flannel + Calico
- Flannel: Networking
- Calico: Network policies

Best of both worlds:
- Simple networking (Flannel)
- Strong policies (Calico)
```

### CNI Plugin Comparison

| Plugin | Type | Performance | Network Policies | Complexity | Use Case |
|--------|------|-------------|------------------|------------|----------|
| **Calico** | L3 (BGP) | High | Excellent | Medium | Production, large scale |
| **Flannel** | Overlay | Medium | Limited | Low | Simple deployments |
| **Cilium** | eBPF | Very High | Excellent (L7) | High | High-performance, modern |
| **Weave** | Mesh | Medium | Good | Low | Multi-cloud, encrypted |
| **Canal** | Hybrid | Medium | Excellent | Medium | Best of Flannel + Calico |

### How CNI Handles Cross-Node Traffic

**Scenario**: Pod on Node1 → Pod on Node2

**Calico (BGP routing)**:
```
1. Pod1 (10.244.1.5) sends to Pod2 (10.244.2.8)
2. Node1 checks routing table:
   10.244.2.0/24 via 10.0.1.20 (Node2 IP)
3. Packet sent directly to Node2 (no encapsulation)
4. Node2 receives, routes to Pod2
5. Native IP routing (fast)
```

**Flannel (VXLAN overlay)**:
```
1. Pod1 (10.244.1.5) sends to Pod2 (10.244.2.8)
2. Node1 encapsulates:
   [Node1 IP][UDP][VXLAN][Pod1→Pod2 packet]
3. Sends to Node2 via VXLAN tunnel
4. Node2 decapsulates:
   Extracts: [Pod1→Pod2 packet]
5. Routes to Pod2
6. Overlay networking (slightly slower, more compatible)
```

**Cilium (eBPF)**:
```
1. Pod1 sends to Pod2
2. eBPF program intercepts at kernel
3. Fast path routing decision
4. Encapsulates if needed (VXLAN or direct)
5. Bypasses iptables completely
6. Extremely fast
```

---

## 8. Common Networking Commands

```bash
# List services
kubectl get svc

# Describe service (see endpoints)
kubectl describe svc my-service

# List pods with IPs
kubectl get pods -o wide

# Check service endpoints (actual Pod IPs)
kubectl get endpoints my-service

# Test connectivity from Pod
kubectl exec -it my-pod -- curl http://other-service

# Port-forward to local machine
kubectl port-forward svc/my-service 8080:80
# Access: http://localhost:8080

# Check DNS resolution
kubectl exec -it my-pod -- nslookup my-service

# List ingresses
kubectl get ingress

# Describe ingress
kubectl describe ingress my-ingress

# List network policies
kubectl get networkpolicies

# Check CNI plugin status
kubectl get pods -n kube-system | grep calico
kubectl get pods -n kube-system | grep flannel
kubectl get pods -n kube-system | grep cilium
```

---

## 9. Troubleshooting

### Issue: Can't Reach Service

**Check if service exists**:
```bash
kubectl get svc my-service
```

**Check if endpoints exist**:
```bash
kubectl get endpoints my-service
# Should show Pod IPs
```

**Check Pod labels match service selector**:
```bash
kubectl get pods --show-labels
kubectl describe svc my-service
# Labels should match
```

### Issue: External Access Not Working

**Check service type**:
```bash
kubectl get svc
# Type should be LoadBalancer or NodePort
```

**Check external IP**:
```bash
kubectl get svc my-service
# Wait for EXTERNAL-IP (may take minutes)
```

**For NodePort, use node IP**:
```bash
kubectl get nodes -o wide
# Use any node IP with NodePort
```

### Issue: Ingress Not Working

**Check ingress controller is running**:
```bash
kubectl get pods -n ingress-nginx
# Should see ingress controller Pods
```

**Check ingress has address**:
```bash
kubectl get ingress
# Should show ADDRESS
```

**Check ingress rules**:
```bash
kubectl describe ingress my-ingress
```

### Issue: Pod Can't Reach Internet

**Check CNI plugin**:
```bash
kubectl get pods -n kube-system
# Ensure CNI pods are Running
```

**Check DNS**:
```bash
kubectl exec -it my-pod -- cat /etc/resolv.conf
# Should have nameserver
```

**Check Network Policies**:
```bash
kubectl get networkpolicy
# Ensure no deny-all policies blocking egress
```

---

## 10. Best Practices

### 1. Use Services, Not Pod IPs
❌ **Bad**: Connect to Pod IP directly  
✅ **Good**: Connect via Service name

### 2. Use DNS Names
```yaml
# Good
BACKEND_URL: http://backend:8080

# Not
BACKEND_URL: http://10.96.0.10:8080
```

### 3. Implement Network Policies
```yaml
# Start with deny-all, then allow specific traffic
# Principle of least privilege
```

### 4. Use Ingress for HTTP(S)
```yaml
# One Ingress with rules
# Better than multiple LoadBalancer Services
```

### 5. Use Health Checks
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

### 6. Use Separate Namespaces
```yaml
# Separate dev, staging, prod
# Prevents accidental cross-environment access
```

### 7. Monitor Network Metrics
```bash
# Check kube-proxy metrics
kubectl top pods -n kube-system

# Check CNI metrics
# Most CNI plugins expose Prometheus metrics
```

---

## 11. Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│                    KEY TAKEAWAYS                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Each Pod gets a unique IP address                     │
│                                                         │
│  Services provide stable DNS names and IPs             │
│                                                         │
│  ClusterIP for internal, LoadBalancer for external     │
│                                                         │
│  Ingress routes HTTP(S) traffic to Services            │
│                                                         │
│  DNS resolution: service-name.namespace.svc.cluster.local│
│                                                         │
│  Network Policies control Pod-to-Pod traffic           │
│                                                         │
│  CNI plugins handle actual networking                  │
│                                                         │
│  Use Service names, not Pod IPs                        │
│                                                         │
│  Port-forward for debugging: kubectl port-forward      │
│                                                         │
│  Always implement Network Policies in production       │
│                                                         │
│  Calico for large clusters, Flannel for simple setups  │
│                                                         │
│  Cilium for high performance and eBPF features         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Reference Cheat Sheet

| Concept | Details |
|---------|---------|
| **Pod IP** | Unique IP per Pod, ephemeral |
| **Service** | Stable IP/DNS for Pods |
| **ClusterIP** | Internal service only |
| **NodePort** | Exposes on node IP (30000-32767) |
| **LoadBalancer** | Cloud provider external LB |
| **Ingress** | Layer 7 routing (HTTP/HTTPS) |
| **CNI** | Container Network Interface plugin |
| **Calico** | BGP routing, high performance |
| **Flannel** | VXLAN overlay, simple |
| **Cilium** | eBPF, high performance, L7 |
| **Network Policy** | Firewall for Pods |
| **kube-proxy** | Manages Service routing |
| **Endpoints** | List of Pod IPs backing Service |

### Service Type Selection Guide

| Need | Recommended Type |
|------|------------------|
| Internal communication | ClusterIP |
| External access (cloud) | LoadBalancer |
| External access (on-prem) | NodePort |
| HTTP routing | Ingress |
| External DNS only | ExternalName |

---

## Practice Problems

Try these to test your understanding:

1. **What is the difference between Pod IP and Service ClusterIP?**
2. **How does kube-proxy implement load balancing for Services?**
3. **What is the purpose of a CNI plugin in Kubernetes?**
4. **Explain the difference between ClusterIP and NodePort services.**
5. **How does Ingress differ from a LoadBalancer Service?**
6. **What is the role of Network Policies in Kubernetes?**
7. **How do you troubleshoot if a Pod cannot reach a Service?**
8. **What are the advantages of using Calico over Flannel?**
9. **How does DNS resolution work for Kubernetes Services?**
10. **What is the difference between iptables and IPVS modes in kube-proxy?**
