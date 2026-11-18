# TCP vs UDP

---

## 1. Quick Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | ✅ Guaranteed delivery | ❌ Best-effort |
| Ordering | ✅ In-order delivery | ❌ May arrive out of order |
| Error checking | ✅ Checksum + retransmit | ✅ Checksum only (no retransmit) |
| Flow control | ✅ Sliding window | ❌ |
| Congestion control | ✅ Slow start, AIMD | ❌ |
| Speed | Slower (overhead) | Faster |
| Header size | 20 bytes min | 8 bytes |
| Use cases | HTTP, SSH, DB, email | DNS, video streaming, gaming, VoIP |

---

## 2. TCP Three-Way Handshake

```
Client                          Server
  |                               |
  |── SYN (seq=x) ──────────────► |   Client: I want to connect, my ISN is x
  |                               |
  |◄── SYN-ACK (seq=y, ack=x+1) ─|   Server: OK, my ISN is y, got your x
  |                               |
  |── ACK (ack=y+1) ─────────────►|   Client: Got your y, connection established
  |                               |
  |════════ DATA TRANSFER ════════|
```

**ISN** = Initial Sequence Number (randomly chosen, prevents packet injection attacks).

### TCP Four-Way Termination (Full Close)
```
Client                          Server
  |── FIN ───────────────────────►|   Client: Done sending
  |◄── ACK ────────────────────── |   Server: OK
  |◄── FIN ────────────────────── |   Server: I'm also done
  |── ACK ───────────────────────►|   Client: OK — TIME_WAIT starts
  |                               |
  (Client stays in TIME_WAIT for 2×MSL = ~60-120s)
```

**TIME_WAIT:** Ensures the last ACK was received; absorbs delayed packets from old connection.

---

## 3. TCP Sequence Numbers & Acknowledgments

```
Sequence number: byte offset of first byte in segment
ACK number: next byte expected from the other side

Example — sending "Hello World" (11 bytes):
  Segment 1: seq=100, data="Hello " (6 bytes) → ack from server: 106
  Segment 2: seq=106, data="World" (5 bytes)  → ack from server: 111
```

**Cumulative ACK:** ACK=106 means "received all bytes up to 105, expecting 106 next."

---

## 4. Flow Control — Sliding Window

**Problem:** Receiver might be overwhelmed by fast sender.
**Solution:** Receiver advertises a **receive window** (rwnd) — how many bytes it can buffer.

```
Sender can have at most rwnd bytes "in flight" (sent but not yet ACKed)

Receive window shrinks as receiver's buffer fills, grows as app reads data.
Window size 0 → sender pauses (zero-window probing to detect recovery)
```

---

## 5. Congestion Control

**Problem:** Too many packets in network → router drops packets → retransmits → more congestion.

### Slow Start
```
cwnd (congestion window) starts at 1 MSS
Every RTT: cwnd doubles (exponential growth)
Until cwnd hits ssthresh (slow start threshold)
```

### Congestion Avoidance (AIMD)
```
After ssthresh: cwnd += 1 MSS per RTT (linear/additive increase)
On packet loss: ssthresh = cwnd/2, cwnd = 1 (multiplicative decrease)
```

### Fast Retransmit / Fast Recovery
```
On 3 duplicate ACKs (not timeout): retransmit immediately
Fast recovery: ssthresh = cwnd/2, cwnd = ssthresh (not back to 1)
```

---

## 6. TCP Nagle's Algorithm

**Problem:** Small packets (1 byte) are inefficient (20-byte TCP header + 20-byte IP header for 1 byte of data).

**Nagle's algorithm:** Buffer small data; send only when ACK received for previous data OR buffer reaches MSS.

**Disable with:** `TCP_NODELAY` socket option (useful for real-time apps like games, SSH).

---

## 7. UDP Use Cases

| Application | Why UDP |
|-------------|---------|
| DNS | Single request-response; retransmit at app layer if needed |
| Video streaming | Stale frames useless; prefer fresh frame over delayed old one |
| Online gaming | Low latency critical; app handles lost updates |
| VoIP | Audio must play in real-time; dropped packet = brief silence |
| DHCP | Broadcast-based; no established connection yet |
| QUIC (HTTP/3) | UDP + reliability at app layer with better performance |

---

## 8. TCP State Machine (Key States)

```
CLOSED → SYN_SENT → SYN_RCVD → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
                               ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED
```

**Check active connections:**
```bash
netstat -an | grep ESTABLISHED
ss -tn state established
```

---

## 9. Quick Reference

| Question | Answer |
|----------|--------|
| Why random ISN? | Prevent TCP hijacking — attacker can't guess sequence numbers |
| What is MSS? | Maximum Segment Size — max TCP payload (usually 1460 bytes = 1500 MTU - 40 IP+TCP headers) |
| What is Nagle's algorithm? | Buffer small packets; reduces overhead for chatty applications |
| TCP vs TLS? | TCP = transport reliability; TLS = encryption/authentication layer on top of TCP |
| What is a half-open connection? | SYN received, SYN-ACK sent, but no ACK — exploited in SYN flood DDoS |
| SYN cookies? | Server-side defense: encodes state in ISN, doesn't store half-open connections |
