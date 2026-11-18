# OSI Model & TCP/IP Stack

---

## 1. OSI vs TCP/IP

| OSI Layer | TCP/IP Layer | PDU | Protocols/Examples |
|-----------|-------------|-----|-------------------|
| 7. Application | Application | Data | HTTP, HTTPS, DNS, FTP, SMTP, SSH |
| 6. Presentation | Application | Data | SSL/TLS, JPEG, ASCII encoding |
| 5. Session | Application | Data | RPC, NetBIOS |
| 4. Transport | Transport | Segment | TCP, UDP |
| 3. Network | Internet | Packet | IP (IPv4/IPv6), ICMP, OSPF, BGP |
| 2. Data Link | Network Access | Frame | Ethernet, Wi-Fi (802.11), ARP |
| 1. Physical | Network Access | Bit | Cable, fiber, radio waves |

**PDU** = Protocol Data Unit (what that layer calls its chunk of data).

---

## 2. Each Layer's Job

### Layer 7 — Application
Provides network services to applications. HTTP defines how web pages are requested and served; DNS resolves names to IPs; SMTP sends email.

### Layer 4 — Transport
End-to-end communication between processes (identified by port numbers).
- **TCP:** Reliable, ordered, connection-oriented
- **UDP:** Unreliable, unordered, connectionless, low-overhead

### Layer 3 — Network
Logical addressing (IP addresses) and routing between networks. Routers operate here.

### Layer 2 — Data Link
Physical addressing (MAC addresses) and frame delivery within a single network segment. Switches operate here.

---

## 3. Encapsulation / Decapsulation

```
Sending side (top to bottom):
  HTTP data → [TCP header | HTTP data] → [IP header | TCP | HTTP] 
           → [Eth header | IP | TCP | HTTP | Eth trailer] → bits

Receiving side (bottom to top):
  bits → strip Eth header/trailer → strip IP header → strip TCP header → HTTP data
```

---

## 4. Important Port Numbers

| Port | Protocol | Service |
|------|----------|---------|
| 20, 21 | TCP | FTP (data, control) |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 143 | TCP | IMAP |
| 443 | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 27017 | TCP | MongoDB |

---

## 5. IP Addressing

### IPv4
- 32-bit address: `192.168.1.100`
- Classes A/B/C (legacy) → replaced by CIDR
- CIDR notation: `192.168.1.0/24` → first 24 bits = network, last 8 = host (256 addresses)

### Private Address Ranges
```
10.0.0.0/8         Class A: 16M hosts
172.16.0.0/12      Class B: 1M hosts
192.168.0.0/16     Class C: 65K hosts
127.0.0.0/8        Loopback
```

### IPv6
- 128-bit: `2001:0db8:85a3::8a2e:0370:7334`
- No NAT needed (enough addresses for every grain of sand)
- Stateless Address Autoconfiguration (SLAAC)

---

## 6. ARP — Address Resolution Protocol

**Problem:** You have a destination IP address. How do you find the MAC address to put in the Ethernet frame?

```
ARP Request (broadcast): "Who has IP 192.168.1.5? Tell 192.168.1.1"
ARP Reply (unicast):     "192.168.1.5 is at MAC AA:BB:CC:DD:EE:FF"
Result: ARP cache: 192.168.1.5 → AA:BB:CC:DD:EE:FF
```

ARP cache entries expire (typically 20 minutes). **Gratuitous ARP:** Unsolicited announcement (used in failover, IP conflict detection).

---

## 7. ICMP

Used for diagnostics and error reporting. Not a transport protocol.

- `ping`: sends ICMP Echo Request, receives Echo Reply → measures RTT
- `traceroute`: sends packets with increasing TTL; each router decrements TTL; at TTL=0 sends "Time Exceeded" → reveals path

```
$ ping google.com
$ traceroute google.com   # Linux
$ tracert google.com      # Windows
```

---

## 8. Interview Quick Reference

| Question | Answer |
|----------|--------|
| What layer does a switch operate on? | Layer 2 (uses MAC addresses) |
| What layer does a router operate on? | Layer 3 (uses IP addresses) |
| Difference between MAC and IP address? | MAC = physical hardware ID (stays with NIC); IP = logical, assigned by network |
| What is a broadcast domain? | Set of devices that receive a Layer 2 broadcast frame; a router separates broadcast domains |
| What is VLAN? | Virtual LAN — logically segments a network at Layer 2 |
| What is NAT? | Network Address Translation — maps private IPs to one public IP; allows multiple devices to share one IP |
