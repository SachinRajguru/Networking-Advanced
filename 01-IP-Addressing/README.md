
# IP Addressing & Subnetting

---

## 1. What is an IP Address?

An **IP (Internet Protocol) address** is a unique numerical label assigned to each device connected to a computer network. It functions like a postal address for computers, enabling devices to find and communicate with each other.

### Primary Functions

| Function | Description |
|----------|-------------|
| **Identification** | Uniquely identifies a device (host or network interface) |
| **Location Addressing** | Provides the location of the device in the network |

---

## 2. How IP Addressing Works

When data travels across networks, it is broken into **packets**. Each packet contains:

```
┌─────────────────────────────────────────┐
│            Network Packet               │
├─────────────────────────────────────────┤
│  Source IP Address      │ Where the packet came from  │
├─────────────────────────┼────────────────────────────┤
│  Destination IP Address │ Where the packet is going   │
├─────────────────────────┼────────────────────────────┤
│  Payload                │ Actual data being transmitted │
└─────────────────────────┴────────────────────────────┘
```

**Routers** use IP addresses to determine the best path for packets to reach their destination.

---

## 3. The Binary Nature of IP Addresses

IP addresses are binary numbers that humans read in **decimal format** for convenience.

### Example: `192.168.1.1` in Binary

```
192     .     168      .      1       .      1
11000000.10101000.00000001.00000001
  ↓        ↓        ↓        ↓
  8 bits   8 bits   8 bits   8 bits
  (octet) (octet) (octet) (octet)
```

### Key Points

| Term | Value |
|------|-------|
| **Octet** | 8 bits = 1 byte |
| **Minimum octet value** | 0 (00000000 in binary) |
| **Maximum octet value** | 255 (11111111 in binary) |
| **Total bits in IPv4** | 32 bits |

> **Important**: Understanding binary is crucial for subnetting and calculating network masks.

---

## 4. Types of IP Addresses

### 4.1 IPv4 (IPv4 Address)

| Property | Value |
|----------|-------|
| **Format** | `192.168.1.1` |
| **Structure** | 4 decimal numbers separated by dots |
| **Range** | 0.0.0.0 to 255.255.255.255 |
| **Total Addresses** | ~4.3 billion |

**Example**: `172.16.0.10`, `10.0.0.1`, `192.168.1.100`

---

### 4.2 IPv6 (IPv6 Address)

| Property | Value |
|----------|-------|
| **Format** | `2001:0db8:85a3:0000:0000:8a2e:0370:7334` |
| **Structure** | 8 groups of hexadecimal numbers |
| **Address Size** | 128 bits (vs 32 bits in IPv4) |
| **Total Addresses** | ~340 undecillion (3.4 × 10³⁸) |

**Why IPv6?**
- IPv4 addresses are exhausted
- IPv6 provides better security
- Built-in auto-configuration
- Better performance

**Examples**:
```
Full:    2001:0db8:85a3:0000:0000:8a2e:0370:7334
Compressed: 2001:db8:85a3::8a2e:370:7334
Link-local: fe80::1
```

---

## 5. IPv4 Address Classes

| Class | First Octet Range | Default Subnet Mask | Use Case | Networks | Hosts per Network |
|-------|-------------------|---------------------|----------|----------|-------------------|
| **A** | 1.0.0.0 – 126.255.255.255 | /8 (255.0.0.0) | Large organizations | 126 | 16,777,214 |
| **B** | 128.0.0.0 – 191.255.255.255 | /16 (255.255.0.0) | Medium organizations | 16,384 | 65,534 |
| **C** | 192.0.0.0 – 223.255.255.255 | /24 (255.255.255.0) | Small networks | 2,097,152 | 254 |
| **D** | 224.0.0.0 – 239.255.255.255 | N/A | Multicast | N/A | N/A |
| **E** | 240.0.0.0 – 255.255.255.255 | N/A | Reserved (experimental) | N/A | N/A |

### Important Notes

```
┌────────────────────────────────────────────────────────┐
│  Special Addresses                                     │
├────────────────────────────────────────────────────────┤
│  0.0.0.0       → Default route (this network)          │
│  127.0.0.1     → Loopback address (localhost)          │
│  255.255.255.255 → Broadcast address                   │
└────────────────────────────────────────────────────────┘
```

---

## 6. Private vs Public IP Addresses

### 6.1 Private IP Addresses

Used **inside** local networks (home, office). Not reachable from the internet.

| CIDR Block | Range | Total Addresses |
|------------|-------|-----------------|
| 10.0.0.0/8 | 10.0.0.0 – 10.255.255.255 | 16,777,216 |
| 172.16.0.0/12 | 172.16.0.0 – 172.31.255.255 | 1,048,576 |
| 192.168.0.0/16 | 192.168.0.0 – 192.168.255.255 | 65,536 |

**Example**: Your laptop at home → `192.168.1.5` (only works within your home network)

---

### 6.2 Public IP Addresses

- **Unique** across the entire internet
- Assigned by **ISP** (Internet Service Provider)
- Required for internet communication

**Examples**:
```
Google DNS:     8.8.8.8
Cloudflare:    1.1.1.1
Google Public: 142.250.190.46
```

---

## 7. Subnetting Basics

**Subnetting** is the process of dividing a larger network into smaller, manageable subnetworks (subnets).

### Why Subnetting?

| Benefit | Description |
|---------|-------------|
| **Efficient IP Allocation** | Prevents wasting addresses |
| **Network Segmentation** | Separates departments/services |
| **Improved Performance** | Smaller broadcast domains = less congestion |
| **Enhanced Security** | Isolates sensitive systems |
| **Simplified Management** | Easier to troubleshoot |

---

## 8. Understanding Subnet Masks

A **subnet mask** is a 32-bit number that divides an IP address into **network** and **host** portions.

### How It Works

```
Bit = 1  → Network portion
Bit = 0  → Host portion
```

### Example: IP `192.168.1.100` with Mask `255.255.255.0`

```
┌─────────────────────────────────────────────────────────────────┐
│  IP Address:       192.168.1.100    11000000.10101000.00000001.01100100 │
│  Subnet Mask:      255.255.255.0    11111111.11111111.11111111.00000000 │
│                                    ════════════════   ═══════════   │
│                                    Network (24 bits)  Host (8)     │
└─────────────────────────────────────────────────────────────────┘
```

### Key Definitions

| Term | Definition | Example (192.168.1.0/24) |
|------|------------|--------------------------|
| **Network Address** | First IP (all host bits = 0) | 192.168.1.0 |
| **Broadcast Address** | Last IP (all host bits = 1) | 192.168.1.255 |
| **First Usable IP** | Network Address + 1 | 192.168.1.1 |
| **Last Usable IP** | Broadcast Address - 1 | 192.168.1.254 |

---

## 9. Common Subnet Masks

### Quick Reference Table

| CIDR | Subnet Mask | Total Addresses | Usable Hosts | Power of 2 |
|------|-------------|-----------------|--------------|------------|
| /32 | 255.255.255.255 | 1 | 0 (single host) | 2⁰ |
| /31 | 255.255.255.254 | 2 | 0 (point-to-point) | 2¹ |
| /30 | 255.255.255.252 | 4 | 2 | 2² |
| /29 | 255.255.255.248 | 8 | 6 | 2³ |
| /28 | 255.255.255.240 | 16 | 14 | 2⁴ |
| /27 | 255.255.255.224 | 32 | 30 | 2⁵ |
| /26 | 255.255.255.192 | 64 | 62 | 2⁶ |
| /25 | 255.255.255.128 | 128 | 126 | 2⁷ |
| **/24** | **255.255.255.0** | **256** | **254** | **2⁸** |
| /23 | 255.255.254.0 | 512 | 510 | 2⁹ |
| /22 | 255.255.252.0 | 1,024 | 1,022 | 2¹⁰ |
| /21 | 255.255.248.0 | 2,048 | 2,046 | 2¹¹ |
| /20 | 255.255.240.0 | 4,096 | 4,094 | 2¹² |
| /19 | 255.255.224.0 | 8,192 | 8,190 | 2¹³ |
| /18 | 255.255.192.0 | 16,384 | 16,382 | 2¹⁴ |
| /17 | 255.255.128.0 | 32,768 | 32,766 | 2¹⁵ |
| **/16** | **255.255.0.0** | **65,536** | **65,534** | **2¹⁶** |
| /8 | 255.0.0.0 | 16,777,216 | 16,777,214 | 2²⁴ |

### Formula

```
Usable Hosts = 2^(32 - CIDR) - 2
```

> **Note**: We subtract 2 because:
> - Network address (all host bits = 0)
> - Broadcast address (all host bits = 1)

---

## 10. CIDR Notation

**CIDR** (Classless Inter-Domain Routing) was introduced in **1993** to replace classful addressing.

### Why CIDR?

| Classful Problem | CIDR Solution |
|------------------|---------------|
| Class C = only 254 hosts (too small) | Flexible prefix length |
| Class B = 65,534 hosts (too large) | Any host count possible |
| Wasted addresses | Efficient allocation |

### CIDR Notation Explained

```
192.168.1.0/24
      ↑      ↑
   IP    Network bits (remaining = host bits)
```

| CIDR | Network Bits | Host Bits | Addresses |
|------|--------------|-----------|-----------|
| /24 | 24 | 8 | 256 |
| /16 | 16 | 16 | 65,536 |
| /8 | 8 | 24 | 16,777,216 |
| /32 | 32 | 0 | 1 |

---

## 11. CIDR Calculation Examples

### Example 1: 10.0.0.0/8

```
Network bits:     8
Host bits:         32 - 8 = 24
Addresses:         2^24 = 16,777,216
Subnet mask:       255.0.0.0
Range:             10.0.0.0 - 10.255.255.255
Usable hosts:     16,777,214
```

---

### Example 2: 172.16.0.0/12

```
Network bits:     12
Host bits:        32 - 12 = 20
Addresses:        2^20 = 1,048,576
Subnet mask:      255.240.0.0
Range:            172.16.0.0 - 172.31.255.255
Usable hosts:     1,048,574
```

---

### Example 3: 192.168.1.0/26

```
Network bits:     26
Host bits:        32 - 26 = 6
Addresses:        2^6 = 64
Usable hosts:     64 - 2 = 62
Subnet mask:      255.255.255.192

┌─────────────────────────────────────────────┐
│  Subnet Breakdown for 192.168.1.0/26       │
├──────────────┬───────────────┬──────────────┤
│ Subnet       │ Range         │ Broadcast    │
├──────────────┼───────────────┼──────────────┤
│ .0           │ .1 - .62      │ .63          │
└──────────────┴───────────────┴──────────────┘
```

---

### Example 4: Subnet 192.168.1.0/24 into /26 subnets

```
Network: 192.168.1.0/24 (256 addresses)

Split into 4 equal /26 subnets:

┌─────────────┬──────────────────┬──────────────┬──────────────┐
│   Subnet    │    Network       │  Usable IP   │  Broadcast   │
├─────────────┼──────────────────┼──────────────┼──────────────┤
│ Subnet 1    │ 192.168.1.0/26   │ .1 - .62     │ 192.168.1.63│
│ Subnet 2    │ 192.168.1.64/26  │ .65 - .126   │ 192.168.1.127│
│ Subnet 3    │ 192.168.1.128/26 │ .129 - .190  │ 192.168.1.191│
│ Subnet 4    │ 192.168.1.192/26 │ .193 - .254  │ 192.168.1.255│
└─────────────┴──────────────────┴──────────────┴──────────────┘
```

---

## 12. Supernetting (Route Aggregation)

**Supernetting** combines multiple networks into a single route to reduce routing table size.

### Example

```
Instead of advertising 4 separate networks:
  • 192.168.1.0/24
  • 192.168.2.0/24
  • 192.168.3.0/24
  • 192.168.4.0/24

Advertise ONE supernet:
  • 192.168.0.0/22

Calculation:
  192.168.00000001.0  → 192.168.1.0
  192.168.00000010.0  → 192.168.2.0
  192.168.00000011.0  → 192.168.3.0
  192.168.00000100.0  → 192.168.4.0
                      ═════════════
              Common prefix: 192.168.0.0/22
```

---

## 13. Real-World Example: AWS VPC Design

```
┌──────────────────────────────────────────────────────────┐
│                    VPC: 10.0.0.0/16                       │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Public Subnet 1   │ 10.0.1.0/24 (Web Servers)      │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Public Subnet 2   │ 10.0.2.0/24 (Load Balancers)   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Private Subnet 1  │ 10.0.10.0/24 (App Servers)     │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Private Subnet 2  │ 10.0.11.0/24 (Internal Apps)   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Database Subnet   │ 10.0.20.0/24 (Databases)       │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

---

## 14. Useful Commands

### Check Your IP Address

```bash
# Linux/Mac
ip addr show
# or
ifconfig

# Windows
ipconfig
```

### Check Network Interface Details

```bash
# Linux - More detailed
ip addr show eth0
ip link show

# Windows
ipconfig /all
```

### Test Connectivity

```bash
# Ping a host
ping 8.8.8.8

# Continuous ping (Linux/Mac)
ping -c 4 8.8.8.8

# Continuous ping (Windows)
ping -n 4 8.8.8.8
```

---

## 15. DevOps Real-World Use Cases

### 15.1 Cloud VPC Design

When setting up cloud infrastructure (AWS, Azure, GCP), you need to design subnets for different tiers:

```
Example: AWS VPC Architecture

┌─────────────────────────────────────────────────────────┐
│                   VPC: 10.0.0.0/16                       │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  Public     │  │  Private    │  │  Database   │     │
│  │  Subnet     │  │  Subnet     │  │  Subnet     │     │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤     │
│  │ Web Servers │  │ App Servers │  │ Databases   │     │
│  │ 10.0.1.0/24 │  │ 10.0.2.0/24 │  │ 10.0.3.0/24 │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
```

### 15.2 Container Networks (Docker)

Docker creates a virtual network for containers:

```bash
# Default Docker network
172.17.0.0/16

# Example container IP
172.17.0.2

# Check Docker networks
docker network ls

# Inspect a network
docker network inspect bridge
```

### 15.3 Kubernetes Pods

Each pod gets its own unique IP address from the cluster network:

```
Kubernetes Cluster Network: 10.244.0.0/16

Pod 1: 10.244.0.2
Pod 2: 10.244.0.3
Pod 3: 10.244.1.2
```

### 15.4 Security Groups & Firewall Rules

Security groups allow traffic from specific IP ranges:

```bash
# AWS Security Group Example
Allow SSH from: 10.0.1.0/24    (Office IP)
Allow HTTP from: 0.0.0.0/0     (Internet)
Allow MySQL from: 10.0.2.0/24  (App tier)

# Kubernetes Network Policy Example
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
spec:
  podSelector:
    matchLabels:
      role: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: app
```

### 15.5 Load Balancers

Load balancers need specific subnets to deploy their nodes:

```
Application Load Balancer (ALB)
├── Subnet 1: 10.0.1.0/24 (Availability Zone A)
└── Subnet 2: 10.0.2.0/24 (Availability Zone B)
```

### 15.6 VPN & Remote Access

VPN servers allocate IP addresses from a defined pool:

```
VPN Subnet: 10.8.0.0/24

Connected clients get:
- Client 1: 10.8.0.2
- Client 2: 10.8.0.3
- Client 3: 10.8.0.4
```

---

## 16. Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│                    KEY TAKEAWAYS                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  IPv4 uses 32-bit addresses (4.3 billion total)        │
│                                                         │
│  IPv6 uses 128-bit addresses (virtually unlimited)     │
│                                                         │
│  Private IPs are for internal networks                │
│                                                         │
│  Public IPs are for internet communication            │
│                                                         │
│  Subnetting organizes and secures networks             │
│                                                         │
│  CIDR notation provides flexible address allocation   │
│                                                         │
│  Subnet mask determines network vs host portions       │
│                                                         │
│  Usable hosts = 2^(32-CIDR) - 2                       │
│                                                         │
│  CIDR is used everywhere in cloud and DevOps          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Reference Cheat Sheet

| Concept | Formula/Note |
|---------|--------------|
| **Usable Hosts** | 2^(32 - n) - 2 |
| **Total Addresses** | 2^(32 - n) |
| **/24** | 256 addresses, 254 usable |
| **/16** | 65,536 addresses, 65,534 usable |
| **/8** | 16,777,216 addresses |
| **Private Range** | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 |
| **Loopback** | 127.0.0.1 |

---

## Practice Problems

Try these to test your understanding:

1. **What is the network address of 192.168.50.123/26?**
2. **How many usable hosts in 10.0.0.0/20?**
3. **Subnet 192.168.1.0/24 into 4 equal subnets**
4. **What is the broadcast address of 172.16.0.0/19?**
