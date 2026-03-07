
# Firewalls & Security Groups

---

## 1. What is a Firewall?

A **firewall** is a security system that monitors and controls incoming and outgoing network traffic based on predetermined rules.

A firewall acts as a barrier between trusted internal networks and untrusted external networks (like the internet). The term "firewall" comes from physical firewalls in buildings that prevent fire from spreading.

### The Purpose of Firewalls

**Network Security Fundamentals**:
In networking, security is based on controlling what traffic is allowed:
- **Default Deny**: Block everything, then explicitly allow what's needed
- **Default Allow**: Allow everything, then block specific threats

Firewalls implement the "Default Deny" principle.

**What Firewalls Protect Against**:

| Threat | Protection |
|--------|------------|
| **Unauthorized Access** | Prevent external attackers from reaching internal systems |
| **Port Scanning** | Hide which services are running |
| **Exploitation** | Block traffic to vulnerable services |
| **Data Exfiltration** | Control what data leaves the network |
| **DDoS Attacks** | Rate limit and filter malicious traffic |
| **Malware Communication** | Block malware from calling home |

---

## 2. Firewall Evolution

| Generation | Era | Technology | Description |
|------------|-----|------------|-------------|
| **1st Generation** | 1980s | Packet Filtering | Check each packet independently (IP, Port, Protocol) |
| **2nd Generation** | 1990s | Stateful Inspection | Track connection state, remember outbound connections |
| **3rd Generation** | 2000s | Application Layer | Inspect application data (HTTP, FTP, DNS), Deep Packet Inspection |
| **4th Generation** | 2010s+ | Next-Gen Firewalls | SSL/TLS decryption, User identity, Cloud integration, ML threat detection |

---

## 3. Firewall Concepts

### Stateful vs Stateless

#### Stateless Firewall (Packet Filter)

**How it works**:
```
Each packet evaluated independently:

Inbound Packet:
  From: 203.0.113.5:54321
  To: 192.168.1.10:80
  Protocol: TCP
  
  Check rules:
  Rule 1: Allow TCP to port 80 → MATCH → ALLOW
```

**Problem**:
```
You need TWO rules:

Rule 1 (Inbound): Allow TCP from anywhere to port 80
Rule 2 (Outbound): Allow TCP from port 80 to anywhere

Without Rule 2, server can't respond!
```

| Characteristics | Description |
|-----------------|-------------|
| **Speed** | Fast (simple comparison) |
| **Memory** | No memory of connections |
| **Rules** | Requires rules for both directions |
| **Security** | Can't distinguish legitimate responses from attacks |

#### Stateful Firewall

**How it works**:
```
Tracks connection state:

1. Outbound request:
   Your PC:54321 → Web Server:443
   Firewall remembers: "Connection from 192.168.1.10:54321 to 93.184.216.34:443"
   
2. Inbound response:
   Web Server:443 → Your PC:54321
   Firewall checks: "Is this a response to an existing connection?"
   Yes → ALLOW (even without explicit rule)
```

**State Table Example**:
```
Connection Table:
[192.168.1.10:54321 ↔ 93.184.216.34:443] State: ESTABLISHED
[192.168.1.10:54322 ↔ 1.1.1.1:53]         State: ESTABLISHED
[192.168.1.10:54323 ↔ 10.0.0.5:3306]      State: ESTABLISHED
```

| Advantages | Description |
|------------|-------------|
| **Smarter Security** | Understands context |
| **Fewer Rules** | Automatically allows legitimate return traffic |
| **Detection** | Can detect connection hijacking |
| **TCP States** | Understands TCP connection states |

**Connection States**:
- **NEW**: New connection attempt
- **ESTABLISHED**: Active connection
- **RELATED**: Related to existing connection (e.g., FTP data channel)
- **INVALID**: Malformed or unexpected packet

**Stateful Rule Example**:
```
# One rule is enough:
Allow outbound to port 443

Firewall automatically:
- Tracks outbound connection
- Allows return traffic
- Drops unsolicited inbound traffic
```

---

### Inbound vs Outbound Rules

#### Inbound (Ingress) Rules
Traffic coming TO your server/network from outside.

**Example Scenario**:
```
External Client → Your Web Server

Inbound Rule Needed:
- Allow TCP port 80 (HTTP)
- Allow TCP port 443 (HTTPS)
- Allow TCP port 22 from specific IP (SSH)
```

**Default Policy**: Usually DENY (for security)

#### Outbound (Egress) Rules
Traffic going FROM your server/network to outside.

**Example Scenario**:
```
Your Server → External API
Your Server → Update servers
Your Server → Database
```

**Default Policy**: Often ALLOW (to not break applications)

**Why Control Outbound?**

| Reason | Description |
|--------|-------------|
| **Data Exfiltration** | Prevent stolen data from leaving |
| **Malware Communication** | Block malware from calling command & control servers |
| **Compliance** | Some regulations require outbound filtering |
| **Cost** | In cloud, outbound bandwidth costs money |

**Strict Outbound Example**:
```
Default: DENY all outbound

Explicit Allow:
- Port 443 to apt.ubuntu.com (package updates)
- Port 443 to specific APIs
- Port 3306 to 10.0.2.5 (internal database)
```

---

## 4. Common Firewall Rules

Firewall rules are evaluated in order. First match wins (in most firewalls).

### Rule Structure

A firewall rule typically contains:

| Field | Description |
|-------|-------------|
| **Priority/Order** | Which rule to evaluate first |
| **Action** | ALLOW or DENY |
| **Protocol** | TCP, UDP, ICMP, or ALL |
| **Source** | IP address or range |
| **Source Port** | Usually ANY |
| **Destination** | IP address or range |
| **Destination Port** | Specific port or range |
| **State** | NEW, ESTABLISHED, RELATED (stateful only) |

**Example Rule**:
```
Priority: 100
Action: ALLOW
Protocol: TCP
Source: 0.0.0.0/0 (anywhere)
Destination: 192.168.1.10 (web server)
Port: 443
State: NEW, ESTABLISHED
```

### Rule Evaluation Order

```
Traffic arrives → Check Rule 1
                 ↓ No match
                Check Rule 2
                 ↓ No match
                Check Rule 3
                 ↓ MATCH → Apply action → DONE
                (Remaining rules not evaluated)
                
If no rules match → Default Policy (usually DENY)
```

**Why Order Matters**:
```
❌ Bad Order:
Rule 1: DENY all from 203.0.113.0/24
Rule 2: ALLOW TCP port 80 from 203.0.113.5

Result: 203.0.113.5 is blocked (Rule 1 matches first)

✓ Good Order:
Rule 1: ALLOW TCP port 80 from 203.0.113.5
Rule 2: DENY all from 203.0.113.0/24

Result: 203.0.113.5 is allowed (Rule 1 matches first)
```

### Common Firewall Rule Patterns

| Pattern | Rule Configuration |
|---------|--------------------|
| **Allow Web Traffic** | Inbound: Port 80, 443 from 0.0.0.0/0 |
| **Allow SSH (Restricted)** | Inbound: Port 22 from specific IP/Subnet |
| **Allow Database (Internal Only)** | Inbound: Port 3306 from App Servers Subnet |
| **Allow All Outbound** | Outbound: All traffic to 0.0.0.0/0 |

---

## 5. Real-World Examples

### Example 1: Web Server Setup

```
┌─────────────────────────────────┐
│ Web Server Security Group       │
├─────────────────────────────────┤
│ Inbound:                        │
│  - Port 80   from 0.0.0.0/0     │
│  - Port 443  from 0.0.0.0/0     │
│  - Port 22   from 1.2.3.4/32    │ ← Your office IP
│                                 │
│ Outbound:                       │
│  - All traffic to 0.0.0.0/0     │
└─────────────────────────────────┘
```

### Example 2: Three-Tier Application

```
Internet → Load Balancer → Web Servers → App Servers → Database

Load Balancer:
  Inbound: 443 from 0.0.0.0/0

Web Servers:
  Inbound: 8080 from Load Balancer only
  Outbound: 5000 to App Servers

App Servers:
  Inbound: 5000 from Web Servers only
  Outbound: 3306 to Database

Database:
  Inbound: 3306 from App Servers only
  Outbound: None (or minimal)
```

### Example 3: Microservices in Kubernetes

```
Frontend Pod:
  - Allow inbound from Ingress Controller
  - Allow outbound to Backend Service

Backend Pod:
  - Allow inbound from Frontend only
  - Allow outbound to Database

Database Pod:
  - Allow inbound from Backend only
  - No external access
```

---

## 6. Linux Firewall (iptables/ufw)

### Using UFW (Ubuntu/Debian - Easy)

```bash
# Enable firewall
sudo ufw enable

# Allow SSH (do this first!)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow specific IP to specific port
sudo ufw allow from 192.168.1.100 to any port 3306

# Allow subnet to port
sudo ufw allow from 10.0.1.0/24 to any port 5432

# Deny specific IP
sudo ufw deny from 203.0.113.0

# Check status
sudo ufw status verbose

# Delete rule
sudo ufw delete allow 80/tcp

# Disable firewall
sudo ufw disable
```

### Using iptables (More Control)

```bash
# List current rules
sudo iptables -L -n -v

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP and HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow from specific IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Block specific IP
sudo iptables -A INPUT -s 203.0.113.0 -j DROP

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Drop everything else
sudo iptables -P INPUT DROP

# Save rules (Ubuntu/Debian)
sudo iptables-save > /etc/iptables/rules.v4
```

---

## 7. AWS Security Groups

### Example: Web Application

```
Security Group: web-servers
├── Inbound Rules:
│   ├── HTTP (80)     Source: 0.0.0.0/0
│   ├── HTTPS (443)   Source: 0.0.0.0/0
│   └── SSH (22)      Source: sg-admin (admin security group)
│
└── Outbound Rules:
    └── All traffic   Destination: 0.0.0.0/0

Security Group: app-servers
├── Inbound Rules:
│   └── Custom TCP (8080)   Source: sg-web-servers
│
└── Outbound Rules:
    └── MySQL (3306)   Destination: sg-database

Security Group: database
├── Inbound Rules:
│   └── MySQL (3306)   Source: sg-app-servers
│
└── Outbound Rules:
    └── None (deny all)
```

---

## 8. Best Practices

### 1. Principle of Least Privilege
Only allow what's necessary.

**Bad**:
```
Allow all traffic from 0.0.0.0/0
```

**Good**:
```
Allow port 443 from 0.0.0.0/0
Allow port 22 from 1.2.3.4/32 (specific IP)
```

### 2. Use Security Groups, Not IP Addresses (Cloud)
Instead of hardcoding IPs, reference other security groups.

```
Database:
  Allow 3306 from sg-app-servers (not from 10.0.1.5)
```

### 3. Document Your Rules
```
Rule: Allow 8080
Purpose: Application API endpoint
Source: Load balancer security group
Added by: John (2024-01-15)
```

### 4. Regular Audits
- Review rules quarterly
- Remove unused rules
- Check for overly permissive rules (0.0.0.0/0)

### 5. Use Separate Security Groups by Role
```
sg-web
sg-app
sg-database
sg-admin
sg-monitoring
```

### 6. Block by Default
Start with "deny all" and explicitly allow what's needed.

---

## 9. Common Firewall Patterns

### Bastion Host (Jump Server)
```
Internet → Bastion (SSH:22 from your IP)
               ↓
           Private Servers (SSH:22 from Bastion only)
```

### DMZ (Demilitarized Zone)
```
Internet → Firewall → DMZ (Public servers)
                      Firewall → Internal Network (Private servers)
```

### Network Segmentation
```
10.0.1.0/24 - Web tier
10.0.2.0/24 - App tier
10.0.3.0/24 - Data tier

Each tier has firewall rules controlling access
```

---

## 10. Troubleshooting

### Issue: Can't Connect to Server

```bash
# 1. Check if port is open locally
sudo netstat -tuln | grep 8080

# 2. Check if firewall is blocking
sudo ufw status
# or
sudo iptables -L -n

# 3. Test from external machine
telnet server-ip 8080
# or
nc -zv server-ip 8080

# 4. Check cloud security groups (AWS/Azure/GCP)
```

### Issue: Connection Times Out vs Connection Refused

| Symptom | Meaning |
|---------|---------|
| **Timeout** | Firewall is blocking (packet dropped) |
| **Refused** | Port is closed, but firewall allows it through |

### Common Mistakes

**Locking yourself out via SSH**:
```bash
# Always allow SSH before enabling firewall!
sudo ufw allow 22
sudo ufw enable
```

**Forgetting to save rules**:
```bash
# Rules are lost on reboot unless saved
sudo iptables-save > /etc/iptables/rules.v4
```

**Opening unnecessary ports**:
```bash
# Don't do this unless you really need it
sudo ufw allow from 0.0.0.0/0 to any
```

---

## 11. Cloud Firewall Commands

### AWS CLI

```bash
# List security groups
aws ec2 describe-security-groups

# Create security group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web servers"

# Add inbound rule
aws ec2 authorize-security-group-ingress \
  --group-id sg-123456 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

---

## 12. Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│                    KEY TAKEAWAYS                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Firewalls control inbound and outbound traffic        │
│                                                         │
│  Default deny, explicitly allow what's needed          │
│                                                         │
│  Use security groups to reference other groups (cloud) │
│                                                         │
│  Separate security groups by role/tier                 │
│                                                         │
│  Always allow SSH before enabling firewall (to avoid lockout)│
│                                                         │
│  Regular audits to remove unnecessary rules            │
│                                                         │
│  Test connectivity after making changes                │
│                                                         │
│  Stateful firewalls are smarter than stateless         │
│                                                         │
│  Document why each rule exists                         │
│                                                         │
│  Network segmentation improves security                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 13. Quick Reference Cheat Sheet

| Concept | Details |
|---------|---------|
| **Firewall** | Controls network traffic based on rules |
| **Default Deny** | Block everything, allow only what's needed |
| **Stateful** | Tracks connection state, smarter security |
| **Stateless** | Evaluates each packet independently |
| **Inbound** | Traffic coming TO your server |
| **Outbound** | Traffic going FROM your server |
| **UFW** | Ubuntu firewall (easy to use) |
| **iptables** | Linux firewall (more control) |
| **Security Groups** | AWS cloud firewall |
| **Bastion Host** | Jump server for accessing private servers |
| **DMZ** | Demilitarized zone for public servers |
| **Connection States** | NEW, ESTABLISHED, RELATED, INVALID |
| **Rule Order** | First match wins |
| **SSH Lockout** | Always allow SSH before enabling firewall |

### Common Ports

| Port | Service | Protocol | Security |
|------|---------|----------|----------|
| 22 | SSH | TCP | Restrict to specific IPs |
| 80 | HTTP | TCP | Allow from anywhere |
| 443 | HTTPS | TCP | Allow from anywhere |
| 3306 | MySQL | TCP | Internal only |
| 5432 | PostgreSQL | TCP | Internal only |
| 6379 | Redis | TCP | Internal only |
| 8080 | HTTP Alt | TCP | Internal or restricted |

### Common CIDR Notation

| CIDR | Description | Use Case |
|------|-------------|----------|
| 0.0.0.0/0 | Anywhere | Public access |
| 10.0.0.0/8 | Private Class A | Internal networks |
| 172.16.0.0/12 | Private Class B | Internal networks |
| 192.168.0.0/16 | Private Class C | Home/Office networks |
| 192.168.1.0/24 | Single subnet | Small network |
| 10.0.1.0/24 | Single subnet | Web tier |
| 10.0.2.0/24 | Single subnet | App tier |
| 10.0.3.0/24 | Single subnet | Database tier |

---

## 14. Practice Problems

Try these to test your understanding:

1. **What is the difference between stateful and stateless firewalls?**
2. **Why is the order of firewall rules important?**
3. **What is the default deny principle and why is it important?**
4. **How do you prevent SSH lockout when enabling a firewall?**
5. **What is a bastion host and when would you use one?**
6. **Explain the difference between inbound and outbound rules.**
7. **Why should you use security groups instead of IP addresses in cloud?**
8. **What are the connection states in a stateful firewall?**
9. **How do you troubleshoot a connection timeout vs connection refused?**
10. **What is network segmentation and how does it improve security?**

---

## 15. Additional Resources

### Learning Resources

| Resource | Type | Description |
|----------|------|-------------|
| **OWASP Firewall Guidelines** | Documentation | Security best practices |
| **NIST Firewall Guidelines** | Documentation | Government security standards |
| **CIS Benchmarks** | Documentation | Security configuration standards |
| **Cloud Security Alliance** | Documentation | Cloud security guidance |

### Tools to Learn

| Tool | Purpose |
|------|---------|
| **Wireshark** | Network packet analysis |
| **Nmap** | Port scanning and security auditing |
| **tcpdump** | Command-line packet capture |
| **ufw** | Ubuntu firewall management |
| **iptables** | Linux firewall configuration |
| **AWS Security Groups** | Cloud firewall management |

### Recommended Reading

- **Network Security Essentials** by William Stallings
- **Firewalls and Internet Security** by W. Richard Stevens
- **Cloud Security for Dummies** by Joseph Muniz
- **AWS Security Best Practices** (AWS Documentation)
