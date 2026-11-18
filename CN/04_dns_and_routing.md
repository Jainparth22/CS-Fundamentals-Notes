# DNS & Routing

---

## 1. DNS Resolution — Step by Step

**Goal:** Resolve `www.example.com` → IP address.

```
User types www.example.com in browser

1. Check browser DNS cache (TTL-based)
2. Check OS DNS cache / /etc/hosts
3. Query Recursive Resolver (usually ISP's or 8.8.8.8)
   ↓
4. Recursive Resolver checks its cache → miss
   ↓
5. Query Root Nameserver (13 sets, e.g., a.root-servers.net)
   → Returns: "I don't know, but .com NS is at a.gtld-servers.net"
   ↓
6. Query TLD Nameserver (a.gtld-servers.net)
   → Returns: "example.com NS is at ns1.example.com (1.2.3.4)"
   ↓
7. Query Authoritative Nameserver (ns1.example.com)
   → Returns: "www.example.com → 93.184.216.34" (A record)
   ↓
8. Recursive Resolver caches result (respecting TTL)
9. Returns IP to client
10. Client caches IP, makes TCP connection
```

**Total time:** 50–100ms (cold), <1ms (cached).

---

## 2. DNS Record Types

| Type | Description | Example |
|------|-------------|---------|
| **A** | IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | IPv6 address | `example.com → 2606:2800::` |
| **CNAME** | Canonical name (alias) | `www.example.com → example.com` |
| **MX** | Mail exchange | `example.com → mail.example.com (priority 10)` |
| **NS** | Nameserver | `example.com → ns1.example.com` |
| **TXT** | Text record | SPF, DKIM, verification |
| **SOA** | Start of Authority | Zone info, primary NS, TTL |
| **PTR** | Reverse DNS (IP → name) | `34.216.184.93.in-addr.arpa → example.com` |
| **SRV** | Service location | `_http._tcp.example.com → server:port` |

**CNAME chain:** `www → example.com → 93.184.216.34` (can't chain CNAME to MX or NS).

---

## 3. DNS Caching & TTL

```
TTL (Time To Live): how long resolvers cache the record
Low TTL (60s):   frequent changes possible, more DNS queries
High TTL (86400s = 24h): stable records, faster resolution (from cache)

For planned IP change: lower TTL days before, change IP, raise TTL after
```

---

## 4. DNS Security

**DNS Spoofing / Cache Poisoning:** Attacker injects false records into resolver cache → redirects traffic.

**DNSSEC:** Digitally signs DNS records with public-key cryptography. Resolver verifies signatures against DNSKEY records.

**DoT / DoH:** DNS over TLS / DNS over HTTPS — encrypts DNS queries to prevent eavesdropping.

---

## 5. Routing Basics

**Routing table:** Each router maintains a table: destination network → next hop.

```bash
# View routing table
ip route show          # Linux
route print            # Windows
netstat -rn            # macOS/Linux

Example:
Destination     Gateway        Iface
0.0.0.0/0       192.168.1.1    eth0    ← default route
192.168.1.0/24  0.0.0.0        eth0    ← directly connected
10.0.0.0/8      192.168.1.254  eth0    ← static route
```

**Longest prefix match:** Router picks the most specific matching route.

---

## 6. Routing Protocols

| Protocol | Type | Algorithm | Use |
|----------|------|-----------|-----|
| **RIP** | Distance Vector | Bellman-Ford | Small networks, legacy |
| **OSPF** | Link State | Dijkstra | Enterprise networks (IGP) |
| **BGP** | Path Vector | Policy-based | Internet backbone (EGP) |
| **EIGRP** | Hybrid | DUAL | Cisco proprietary |

**IGP (Interior Gateway Protocol):** Routing within an AS (Autonomous System) — OSPF, RIP.
**EGP (Exterior Gateway Protocol):** Routing between ASes — BGP.

---

## 7. BGP — The Internet's Routing Protocol

BGP routers exchange reachability information between Autonomous Systems (AS).

```
AS = organization with its own routing policy (ISP, company, cloud provider)
     identified by AS number (ASN), e.g., Google = AS15169

eBGP: between different ASes (over the internet)
iBGP: within the same AS

BGP path selection (simplified):
1. Highest LOCAL_PREF (prefer internal paths)
2. Shortest AS_PATH (fewer hops)
3. Lowest MED (metric from neighbor)
4. eBGP over iBGP
5. Lowest router ID
```

---

## 8. NAT — Network Address Translation

```
Private network: 192.168.1.0/24  ←→  NAT Router ←→  Internet (1 public IP)

Outbound:
  192.168.1.5:54321 → 8.8.8.8:53
  NAT table: 203.0.113.1:1234 → 192.168.1.5:54321
  Packet becomes: 203.0.113.1:1234 → 8.8.8.8:53

Inbound:
  8.8.8.8:53 → 203.0.113.1:1234
  NAT table lookup → translate to 192.168.1.5:54321
```

**NAPT / PAT (Port Address Translation):** Multiple private IPs share one public IP via port mapping. This is what home routers do.

---

## 9. Subnetting Quick Reference

```
/24 → 256 addresses, 254 usable, mask 255.255.255.0
/25 → 128 addresses, 126 usable, mask 255.255.255.128
/26 → 64 addresses, 62 usable, mask 255.255.255.192
/27 → 32 addresses, 30 usable
/28 → 16 addresses, 14 usable
/30 → 4 addresses, 2 usable (point-to-point links)
/32 → 1 host (loopback, specific host route)

Usable = total - 2 (network address + broadcast address)
```

---

## 10. Quick Reference

| Question | Answer |
|----------|--------|
| What is a recursive resolver? | DNS server that does the full lookup on client's behalf (e.g., 8.8.8.8, 1.1.1.1) |
| What is an authoritative nameserver? | Has the actual DNS records for a zone; gives definitive answers |
| What is anycast? | Multiple servers share same IP; routing sends client to nearest (used by DNS root servers, CDNs) |
| Difference between routing and switching? | Switching = Layer 2 (MAC, within LAN); Routing = Layer 3 (IP, between networks) |
| What is DHCP? | Automatically assigns IP, subnet mask, gateway, DNS to clients on the network |
