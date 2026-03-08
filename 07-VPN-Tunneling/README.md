# VPN & Tunneling

---

## 1. What is a VPN?

**VPN** (Virtual Private Network) creates a secure, encrypted connection over a public network (like the internet).

### The Core Problem VPNs Solve

**Scenario**: Company has office in New York and branch in London.

| Option | Description | Cost | Pros | Cons |
|--------|-------------|------|------|------|
| **Dedicated Line** | Physical leased line between offices | $10,000+/month | Secure, private, high bandwidth | Expensive, inflexible, long setup |
| **VPN** | Encrypted tunnel through public internet | $50/month + internet | Cheap, flexible, quick setup | Dependent on internet quality |

**VPN creates "virtual" private network over public infrastructure.**

---

## 2. What VPN Actually Does

### 1. Encapsulation (Tunneling)

```
Original Packet:
[IP Header][TCP Header][Data: "secret message"]

After VPN Encapsulation:
[VPN IP Header][VPN Header][Encrypted:[IP Header][TCP Header][Data: "secret message"]]
     ↑                            ↑
  Outer packet             Inner packet (encrypted)
  (for internet routing)   (actual data, hidden)
```

### 2. Encryption

```
Without VPN:
Your Computer → "username=admin&password=secret" → Website
               (ISP, government, hackers can read this)

With VPN:
Your Computer → VPN: "a7f3b9c2d..." → Website
               (ISP sees encrypted gibberish)
               
VPN Server → "username=admin&password=secret" → Website
            (Decrypted at VPN server)
```

### 3. IP Address Masking

```
Your Real IP: 203.0.113.5 (New York)
              ↓
            VPN Tunnel
              ↓
VPN Server IP: 198.51.100.10 (London)
              ↓
            Website

Website sees: 198.51.100.10 (thinks you're in London)
```

---

## 3. VPN Components

| Component | Description |
|-----------|-------------|
| **VPN Client** | Software on your device that initiates VPN connection |
| **VPN Server** | Endpoint on remote network that accepts VPN connections |
| **VPN Protocol** | Rules for establishing and maintaining connection |
| **VPN Tunnel** | Virtual encrypted connection through public network |

---

## 4. Types of VPNs

### 1. Remote Access VPN

Connects individual users to a private network.

```
Remote Worker (Home)
        |
    [VPN Client]
        |
    Internet
        |
  [VPN Server]
        |
  Company Network
```

**Use case**: Working from home, accessing company resources.

### 2. Site-to-Site VPN

Connects entire networks together.

```
Office A Network
        |
  [VPN Gateway]
        |
    Internet
        |
  [VPN Gateway]
        |
Office B Network
```

**Use case**: Connecting branch offices, connecting on-premise to cloud.

### 3. Cloud VPN

Connects your network to cloud resources.

```
On-Premise Network
        |
  [VPN Gateway]
        |
    Internet
        |
  [Cloud VPN Gateway]
        |
  AWS/Azure/GCP VPC
```

**Use case**: Hybrid cloud architecture.

---

## 5. How VPN Works

```
1. Your Computer
   ↓ (Data is encrypted)
2. VPN Client
   ↓ (Encrypted tunnel through internet)
3. VPN Server
   ↓ (Data is decrypted)
4. Destination (Website/Server)
```

**Without VPN**:
```
Your IP: 203.0.113.5 → Website sees: 203.0.113.5
```

**With VPN**:
```
Your IP: 203.0.113.5 → VPN → Website sees: 198.51.100.10 (VPN server IP)
```

---

## 6. VPN Protocols

### OpenVPN

**Architecture**:
- Open-source (auditable code)
- Uses OpenSSL library for encryption
- Custom protocol built on SSL/TLS
- Highly configurable

**How it works**:
```
1. Client initiates connection to OpenVPN server
2. TLS handshake (like HTTPS):
   - Certificate verification
   - Key exchange
   - Encryption negotiation
3. Virtual network interface created (tun/tap)
4. Traffic routed through encrypted tunnel
```

**Encryption**:
- Supports multiple algorithms: AES-256, ChaCha20, Blowfish
- Perfect Forward Secrecy (PFS): New keys per session
- HMAC authentication: SHA-256, SHA-512

**Transport**:

| Mode | Description | Use Case |
|------|-------------|----------|
| **TCP Mode** | Reliable, slower, works everywhere | Restrictive firewalls |
| **UDP Mode** | Faster, preferred for VPN | VoIP, gaming, streaming |

| Advantages | Disadvantages |
|------------|---------------|
| Most secure (proven track record) | Slower than WireGuard |
| Works on all platforms | Complex configuration |
| Bypasses most firewalls | Requires third-party client software |

---

### WireGuard

**Architecture**:
- Modern, minimalist design
- ~4,000 lines of code (vs OpenVPN's 400,000)
- Built into Linux kernel
- Uses state-of-the-art cryptography

**Cryptography (No Options - Simplicity)**:
- Encryption: ChaCha20
- Authentication: Poly1305
- Key Exchange: Curve25519
- Hash: BLAKE2s

**Why Fixed Cryptography?**:
```
OpenVPN: User chooses algorithm (can choose weak ones)
WireGuard: Only modern, secure algorithms available
Result: Simpler, impossible to misconfigure
```

**Key Features**:

| Feature | Description |
|---------|-------------|
| **Roaming** | Automatically maintains connection when switching networks |
| **Performance** | 10x faster than OpenVPN |
| **Mobile** | Battery efficient |
| **Setup** | Simple configuration |

| Advantages | Disadvantages |
|------------|---------------|
| Extremely fast | Newer (less proven than OpenVPN) |
| Simple configuration | Static IP assignment (privacy concern) |
| Built into Linux kernel (5.6+) | Requires modern systems |
| Strong security by default | |

---

### IPSec (Internet Protocol Security)

**Architecture**:
- Industry standard
- Built into many operating systems
- Suite of protocols, not single protocol
- Complex but powerful

**IPSec Modes**:

| Mode | Description | Use Case |
|------|-------------|----------|
| **Transport Mode** | Only payload encrypted | Host-to-host communication |
| **Tunnel Mode** | Entire original packet encrypted | Site-to-site VPNs, gateway-to-gateway |

**IPSec Components**:

| Component | Description |
|-----------|-------------|
| **AH (Authentication Header)** | Provides integrity and authentication, no encryption |
| **ESP (Encapsulating Security Payload)** | Provides encryption + authentication, most commonly used |
| **IKE (Internet Key Exchange)** | Phase 1: Establish secure channel, Phase 2: Establish IPSec tunnel |

| Advantages | Disadvantages |
|------------|---------------|
| Industry standard (interoperable) | Complex configuration |
| Built into most OSes (no client needed) | Difficult to troubleshoot |
| Strong encryption | NAT traversal issues |
| Widely supported in enterprise | Firewall unfriendly |

---

### SSL/TLS VPN

**How it works**:
```
Uses HTTPS (port 443) for VPN tunnel
- Runs in web browser or thin client
- Leverages SSL/TLS for encryption
- Same technology as secure websites
```

**Two Modes**:

| Mode | Description | Use Case |
|------|-------------|----------|
| **Clientless (Portal)** | Access via browser, no client software | Web-based applications |
| **Client-Based (Tunnel)** | Full network access, requires client software | All applications work |

| Advantages | Disadvantages |
|------------|---------------|
| Works through any firewall (uses port 443) | Slower than IPSec/WireGuard |
| No special client for portal mode | Limited in portal mode |
| Easy to use | Less efficient encapsulation |
| Granular access control | |

---

### Protocol Comparison

| Feature | OpenVPN | WireGuard | IPSec | SSL VPN |
|---------|---------|-----------|-------|---------|
| **Speed** | Medium | Fastest | Fast | Slowest |
| **Security** | Excellent | Excellent | Excellent | Good |
| **Ease of Setup** | Medium | Easy | Difficult | Easy |
| **Platform Support** | All | Most | All | All |
| **Mobile Performance** | Medium | Excellent | Good | Medium |
| **Code Complexity** | High | Very Low | Very High | High |
| **Firewall Friendly** | Yes | Yes | No | Yes |

### When to Use Each

| Protocol | When to Use |
|----------|-------------|
| **OpenVPN** | Need cross-platform compatibility, want proven solution, require flexible configuration |
| **WireGuard** | Need maximum performance, modern infrastructure, mobile users, simple setup preferred |
| **IPSec** | Enterprise site-to-site VPN, vendor equipment requirement, compliance requirements |
| **SSL VPN** | Remote access through restrictive firewalls, browser-based access needed, quick deployment |

---

## 7. Tunneling Concepts

### What is a Tunnel?

**The Concept**:
```
You want to send Protocol A through Network B
Network B doesn't support Protocol A
Solution: Wrap Protocol A packets inside Protocol B packets

Like: Mailing a letter (Protocol A) inside a package (Protocol B)
```

**Packet in Packet**:
```
Original Packet (Inner):
[IP: 10.0.1.5 → 10.0.2.10][TCP][Data]

Tunneled Packet (Outer):
[IP: 203.0.113.5 → 198.51.100.10][Tunnel Protocol][Encrypted:[Inner Packet]]
        ↑                                   ↑                ↑
   Public IPs                        Tunnel Header    Original packet
```

### Why Tunneling?

| Reason | Description |
|--------|-------------|
| **Security** | Encrypt private traffic over public network |
| **Protocol Compatibility** | Send IPv6 over IPv4 network |
| **Network Extension** | Connect remote networks as if local |
| **Bypass Restrictions** | Access blocked services, hide traffic type |

### Tunneling vs Encryption

**Important Distinction**:
```
Tunneling ≠ Encryption

Tunneling: Packet encapsulation (may or may not be encrypted)
Encryption: Making data unreadable

GRE Tunnel: Tunneling without encryption
VPN: Tunneling WITH encryption
```

---

### Common Tunneling Protocols

#### GRE (Generic Routing Encapsulation)

**Purpose**: Simple tunneling protocol, wraps any protocol inside IP

**Packet Structure**:
```
[IP Header: GW1 → GW2][GRE Header][Inner Packet]
```

**Characteristics**:
- No encryption (plaintext)
- Stateless (no session tracking)
- Low overhead
- Often combined with IPSec for security

**Use Case**:
```
Branch Office Network (10.0.0.0/24)
        ↓
    GRE Tunnel (over internet)
        ↓
HQ Network (192.168.0.0/16)

Networks connected as if directly connected
```

#### VXLAN (Virtual Extensible LAN)

**Purpose**: Create virtual Layer 2 networks over Layer 3 infrastructure

**The Problem VXLAN Solves**:
```
Traditional VLANs:
- Limited to 4096 VLANs (12-bit VLAN ID)
- Can't span multiple datacenters
- Tied to physical infrastructure

VXLAN:
- 16 million virtual networks (24-bit VNI)
- Works over IP networks (across datacenters)
- Overlay network (decoupled from physical)
```

**How VXLAN Works**:
```
VM1 (VXLAN 100) on Host A wants to talk to VM2 (VXLAN 100) on Host B

1. VM1 sends Ethernet frame
2. Host A encapsulates in VXLAN:
   [UDP][VXLAN Header: VNI=100][Original Ethernet Frame]
3. Sent to Host B over IP network
4. Host B decapsulates and delivers to VM2

Result: VM1 and VM2 think they're on same L2 network
```

**Use Cases**:
- Multi-tenant cloud environments
- Container networking (Docker, Kubernetes)
- Datacenter network virtualization
- Connecting VMs across locations

#### IP-in-IP

**Purpose**: Encapsulate IP packet in another IP packet

**Structure**:
```
[Outer IP Header][Inner IP Header][TCP][Data]
     ↑                  ↑
  For routing      Original packet
```

**Use Case**:
```
Mobile IP: Device keeps same IP when moving networks
IPv6 over IPv4: Tunnel IPv6 through IPv4 network
```

---

### Tunnel Modes

#### Point-to-Point Tunnel
```
Endpoint A ←-------- Tunnel --------→ Endpoint B

One-to-one connection
Examples: SSH tunnel, PPP, L2TP
```

#### Hub-and-Spoke Tunnel
```
       Branch 1
          ↓
       Tunnel
          ↓
Hub (HQ) ←-- Tunnel --- Branch 2
          ↓
       Tunnel
          ↓
       Branch 3

Multiple branches connect to central hub
```

#### Mesh Tunnel
```
Office A ←--→ Office B
   ↕             ↕
Office C ←--→ Office D

Every office connected to every other
Full redundancy, but complex to manage
```

---

### Split Tunneling

**Full Tunnel** (All traffic through VPN):
```
Your Computer → VPN → Corporate Network
             → VPN → Internet

All traffic protected, but:
- Slower (everything routes through VPN)
- Corporate network sees all traffic
- Corporate bandwidth used for personal traffic
```

**Split Tunnel** (Only corporate traffic through VPN):
```
Your Computer → VPN → Corporate Network (10.0.0.0/8)
             → Direct → Internet (everything else)

Faster, but:
- Less secure (split security boundary)
- Corporate network can't see/control all traffic
- Potential for data leakage
```

**Configuration**:
```
Full Tunnel:
Route: 0.0.0.0/0 via VPN (all traffic)

Split Tunnel:
Route: 10.0.0.0/8 via VPN (only corporate)
Route: 0.0.0.0/0 via local gateway (everything else)
```

**When to Use Each**:

| Type | When to Use |
|------|-------------|
| **Full Tunnel** | Maximum security required, compliance requirements, want to monitor all traffic, accessing sensitive data |
| **Split Tunnel** | Better performance needed, high bandwidth apps (streaming, gaming), trust user network, VPN capacity limited |

---

### Tunnel Overhead

Every tunnel adds overhead:

**Packet Size Calculation**:
```
Original Packet: 1500 bytes (Ethernet MTU)
- IP Header: 20 bytes
- TCP Header: 20 bytes
- Data: 1460 bytes

After Tunneling (VPN):
- Outer IP Header: 20 bytes
- VPN Header: 20-50 bytes (varies by protocol)
- Encrypted Original Packet: 1500 bytes
Total: 1540-1570 bytes

Problem: Exceeds Ethernet MTU!
```

**MTU Issues**:
```
Without proper configuration:
1. Large packet created (1570 bytes)
2. Exceeds network MTU (1500 bytes)
3. Packet fragmented
4. Fragments may be dropped
5. Connection fails or is very slow

Solution:
- Reduce MTU on tunnel interface (1400 bytes)
- Enable Path MTU Discovery
- Configure MSS clamping
```

**Performance Impact**:
```
Overhead per packet:
- No tunnel: 40 bytes (IP+TCP)
- GRE: 64 bytes (IP+GRE+IP+TCP)
- IPSec: 70-100 bytes
- OpenVPN: 80-120 bytes

For small packets, overhead can be 50%+ of packet size
```

---

## 8. Real-World Examples

### Example 1: Remote Work Setup
```
Employee at Home
       ↓
  VPN Client (OpenVPN)
       ↓
    Internet
       ↓
  Company VPN Server
       ↓
Access: File servers, databases, internal apps
```

### Example 2: AWS Site-to-Site VPN
```
On-Premise Data Center (10.0.0.0/16)
       ↓
  Customer Gateway
       ↓
   IPSec Tunnel
       ↓
  AWS Virtual Private Gateway
       ↓
AWS VPC (172.16.0.0/16)
```

### Example 3: Bastion Host with SSH Tunnel
```
Developer
       ↓
SSH Tunnel to Bastion (public IP)
       ↓
Access Database (private IP, no public access)
```

---

## 9. Setting Up a Simple VPN

### WireGuard (Modern & Simple)

**Server Setup**:
```bash
# Install WireGuard
sudo apt install wireguard

# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey

# Configure (/etc/wireguard/wg0.conf)
[Interface]
Address = 10.0.0.1/24
PrivateKey = <server-private-key>
ListenPort = 51820

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32

# Start VPN
sudo wg-quick up wg0

# Enable on boot
sudo systemctl enable wg-quick@wg0
```

**Client Setup**:
```bash
# Configure (/etc/wireguard/wg0.conf)
[Interface]
Address = 10.0.0.2/24
PrivateKey = <client-private-key>

[Peer]
PublicKey = <server-public-key>
Endpoint = vpn-server.example.com:51820
AllowedIPs = 0.0.0.0/0  # Route all traffic through VPN
PersistentKeepalive = 25

# Connect
sudo wg-quick up wg0
```

---

## 10. SSH Tunneling (Port Forwarding)

### Local Port Forwarding
Access remote service through local port.

```bash
# Forward local port 8080 to remote server's port 80
ssh -L 8080:localhost:80 user@remote-server

# Now access: http://localhost:8080
```

**Use case**: Access web interface on remote server.

### Remote Port Forwarding
Expose local service to remote server.

```bash
# Expose local port 3000 on remote server's port 8080
ssh -R 8080:localhost:3000 user@remote-server

# Remote users can access: http://remote-server:8080
```

**Use case**: Demo local development to others.

### Dynamic Port Forwarding (SOCKS Proxy)
```bash
# Create SOCKS proxy
ssh -D 8080 user@remote-server

# Configure browser to use SOCKS proxy: localhost:8080
```

**Use case**: Route all browser traffic through remote server.

---

## 11. Cloud VPN Examples

### AWS VPN
```
Components:
- Virtual Private Gateway (VPC side)
- Customer Gateway (your side)
- VPN Connection (IPSec tunnel)

Configuration:
- Pre-shared key for authentication
- BGP or static routing
- Redundant tunnels for high availability
```

### Azure VPN Gateway
```
Components:
- VPN Gateway in Azure VNet
- Local Network Gateway (your on-premise)
- VPN Connection

Types:
- Point-to-Site (client VPN)
- Site-to-Site (network VPN)
```

### GCP Cloud VPN
```
Components:
- Cloud Router
- VPN Gateway
- Tunnel

Features:
- High availability with multiple tunnels
- BGP routing support
- Integration with Cloud Interconnect
```

---

## 12. VPN vs Proxy vs Tor

| Feature | VPN | Proxy | Tor |
|---------|-----|-------|-----|
| **Encryption** | Yes | Depends | Yes |
| **Speed** | Fast | Fastest | Slow |
| **Privacy** | Good | Limited | Best |
| **Cost** | Paid/Free | Free | Free |
| **Use Case** | General privacy | Bypass filters | Anonymity |
| **Setup** | Easy | Very Easy | Complex |
| **Trust** | VPN Provider | Proxy Server | Multiple Nodes |

---

## 13. Common Commands

### Check VPN connection:
```bash
# List network interfaces
ip addr show
# Look for wg0, tun0, or similar VPN interface

# Check routing
ip route show

# Test if traffic is going through VPN
curl ifconfig.me  # Shows your public IP
```

### WireGuard commands:
```bash
# Start VPN
sudo wg-quick up wg0

# Stop VPN
sudo wg-quick down wg0

# Check status
sudo wg show

# Check interface
ip addr show wg0
```

### OpenVPN commands:
```bash
# Connect to VPN
sudo openvpn --config client.ovpn

# Run in background
sudo systemctl start openvpn@client

# Check status
sudo systemctl status openvpn@client
```

### SSH Tunnel commands:
```bash
# Local port forwarding
ssh -L 8080:localhost:80 user@remote-server

# Remote port forwarding
ssh -R 8080:localhost:3000 user@remote-server

# Dynamic SOCKS proxy
ssh -D 8080 user@remote-server
```

---

## 14. Troubleshooting

### Issue: Can't connect to VPN
```bash
# Check if VPN server is reachable
ping vpn-server.example.com

# Check if port is open
nc -zv vpn-server.example.com 51820

# Check firewall
sudo ufw allow 51820/udp
```

### Issue: Connected but no internet
```bash
# Check DNS
cat /etc/resolv.conf

# Check routing
ip route show

# Ensure IP forwarding is enabled (server)
sudo sysctl -w net.ipv4.ip_forward=1
```

### Issue: Slow VPN speed
- Try UDP instead of TCP (if available)
- Choose closer VPN server
- Check for ISP throttling
- Use WireGuard instead of OpenVPN (faster)

### Issue: MTU Problems
```bash
# Check current MTU
ip link show wg0

# Reduce MTU
sudo ip link set wg0 mtu 1400

# Test with ping
ping -M do -s 1472 vpn-server.example.com
```

---

## 15. DevOps Use Cases

### 1. Secure Access to Private Resources
```
Developer → VPN → Private Database (no public IP)
```

### 2. Hybrid Cloud Connectivity
```
On-Premise Servers → VPN → Cloud Resources
Enables seamless hybrid deployments
```

### 3. Multi-Region Networking
```
US Datacenter → VPN → EU Datacenter → VPN → Asia Datacenter
```

### 4. Secure CI/CD Access
```
CI/CD Server → VPN → Production Environment
Deploy safely without exposing production
```

### 5. Secure API Access
```
Internal Service → VPN → External API Gateway
Protect API endpoints from public access
```

---

## 16. Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│                    KEY TAKEAWAYS                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  VPNs create encrypted tunnels over public networks    │
│                                                         │
│  Remote access VPN for individuals, site-to-site for networks│
│                                                         │
│  WireGuard is modern and fast, OpenVPN is mature and flexible│
│                                                         │
│  SSH tunneling is simple for quick port forwarding     │
│                                                         │
│  Cloud VPNs connect on-premise to cloud (hybrid cloud) │
│                                                         │
│  Split tunneling sends only some traffic through VPN   │
│                                                         │
│  Always use VPN on public WiFi for security            │
│                                                         │
│  Check firewall and routing when troubleshooting VPN issues│
│                                                         │
│  Tunneling adds overhead - configure MTU properly      │
│                                                         │
│  Choose protocol based on use case (speed vs security) │
│                                                         │
│  Full tunnel for security, split tunnel for performance│
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Reference Cheat Sheet

| Concept | Details |
|---------|---------|
| **VPN** | Virtual Private Network - encrypted tunnel over public network |
| **Tunneling** | Encapsulating one protocol inside another |
| **WireGuard** | Modern, fast, simple, built into Linux kernel |
| **OpenVPN** | Mature, flexible, cross-platform, slower |
| **IPSec** | Industry standard, complex, enterprise-focused |
| **SSL VPN** | Browser-based, uses port 443, easy to deploy |
| **Split Tunnel** | Only route specific traffic through VPN |
| **Full Tunnel** | Route all traffic through VPN |
| **SSH Tunnel** | Quick port forwarding for specific services |
| **MTU** | Maximum Transmission Unit - configure for tunnels |
| **GRE** | Generic Routing Encapsulation - no encryption |
| **VXLAN** | Virtual Extensible LAN - Layer 2 over Layer 3 |

### Protocol Selection Guide

| Need | Recommended Protocol |
|------|---------------------|
| Maximum speed | WireGuard |
| Cross-platform compatibility | OpenVPN |
| Enterprise site-to-site | IPSec |
| Browser-based access | SSL VPN |
| Quick testing | SSH Tunnel |
| Container networking | VXLAN |

---

## Practice Problems

Try these to test your understanding:

1. **What is the difference between tunneling and encryption?**
2. **When would you use split tunneling vs full tunneling?**
3. **What are the main differences between WireGuard and OpenVPN?**
4. **How does IPSec tunnel mode differ from transport mode?**
5. **What is the purpose of MTU configuration in VPNs?**
6. **Explain the difference between remote access and site-to-site VPNs.**
7. **How does SSH tunneling work and when would you use it?**
8. **What are the advantages of VXLAN over traditional VLANs?**
9. **How do you troubleshoot a VPN connection that is slow?**
10. **What is the difference between VPN, proxy, and Tor?**
