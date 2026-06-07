# Protocol Analysis Report Condensed
Wireshark Packet Capture Analysis Task 5

## Executive Summary
Analysis of 11,836 packets identified four protocols (DNS, TCP, TLS, ARP) operating across OSI layers. All protocols functioning correctly with no anomalies. Network operating with modern security practices.

## DNS Protocol Analysis
**Fundamentals:** Layer 7 | UDP Port 53 | Maps domain names to IP addresses

**Captured Queries:**
- brave.com: AAAA query (Ipv6) at packet 1360, resolved successfully
- search.brave.com: AAAA query at packet 2703, NOERROR status
- www.imdb.com: A query with CNAME pointing to frontier.imdb.com

**Response Types:**
- AAAA Records: 128-bit Ipv6 addresses (example: 2600:9000:2579:4e00::)
- A Records: 32-bit Ipv4 addresses (example: 18.161.229.79)
- CNAME Records: Aliases for load balancing and CDN routing
- NS Records: Nameserver information (ns-772.awsdns-32.net)

**DNS Packet Structure:**
Query Header: 16-bit transaction ID, flags 0x0100 (standard query), 1 question
Response Flags: 0x8180 (response + recursion available), 2-5 answer records

**Key Observations:**
✓ Dual-stack support (both A and AAAA queries)
✓ CDN routing with cloudfront.net responses
✓ All queries received responses (NOERROR status)
✓ No resolution failures detected
✓ Query IDs properly matched with responses
✓ No duplicate queries (DNS caching working)

---

## TCP Protocol Analysis
**Fundamentals:** Layer 4 | Connection-oriented | Reliable, in-order delivery

**Three-Way Handshake:**
1. Client SYN: Seq=0, window=133 bytes, port 60035 → 443
2. Server SYN-ACK: Seq=1, Ack=1, window=1329 bytes
3. Client ACK: Seq=1, Ack=1
Handshake time: ~300 microseconds

**Data Transmission Mechanics:**
- Sequence numbers increment by bytes transmitted
- Acknowledgment numbers confirm receipt
- PSH flag pushes data to application immediately
- No sequence gaps observed (all bytes accounted for)

**Example Flow:**
Packet 5: Seq=1, Ack=4277 [PSH,ACK] + 86 bytes
Packet 7: Seq=4277, Ack=87 [ACK] (acknowledges 86 bytes)
Packet 11: Seq=87, Ack=4277 [ACK] + 443 bytes

**Connection Termination:**
FIN-ACK → ACK → FIN-ACK → ACK (clean bidirectional close)
Some connections closed with RST flag (abrupt, less common)

**Connection Parameters:**
- Port 443 (HTTPS) with Ipv6 addresses
- Window size: 133-1329 bytes throughout
- MSS: Typical 1440-1500 bytes (observed segments: 86-1174 bytes)

**TCP Flags:**
- SYN: Connection initiation
- ACK: Confirms data receipt
- PSH: Flush buffer to application
- FIN: Graceful close
- RST: Abrupt termination

**Network Health:**
✓ No sequence gaps (reliable delivery confirmed)
✓ No duplicate packets (no retransmissions)
✓ No out-of-order delivery
✓ No window shrinking to zero (no flow control issues)
✓ Multiple concurrent connections on port 443
✓ Healthy RTT (~200ms range)

---

## TLS Protocol Analysis
**Fundamentals:** Layer 6 | Encryption and authentication | Ports 443, 465, etc.

**TLS 1.2 Handshake Sequence (7 Messages, ~42ms total):**

1. **Client Hello** (Packet 2, 1414 bytes)
   - Supported versions: TLS 1.2, 1.3
   - Cipher suite offers
   - 64-byte random nonce

2. **Server Hello** (Packet 3, 86 bytes)
   - Selected TLS 1.2
   - Selected cipher suite: AES with Diffie-Hellman
   - 64-byte server random
   - Session ID for resumption

3. **Certificate** (Packet 8, 1414 bytes)
   - X.509 digital certificate with server's public key
   - Signed by Certificate Authority
   - Certificate chain included

4. **Server Key Exchange** (Packet 10, 138 bytes)
   - Ephemeral Diffie-Hellman public key
   - Signed with server's private key
   - Enables forward secrecy

5. **Client Key Exchange** (Packet 12, 212 bytes)
   - Client's ephemeral DH public key
   - Both parties compute shared secret
   - Session encryption key derived

6. **Change Cipher Spec** (Packet 13, 185 bytes)
   - Signals switch from unencrypted to encrypted

7. **Encrypted Finish** (Packet 14, 509 bytes)
   - MAC of all handshake messages
   - Encrypted with derived session key
   - Verifies handshake integrity

**Encryption Implementation:**
Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- ECDHE: Elliptic Curve DH (key exchange with forward secrecy)
- RSA: Certificate signature algorithm
- AES_128_GCM: 128-bit AES in GCM mode (authenticated encryption)
- SHA256: HMAC for integrity

Application Data: 1294-1414 bytes encrypted payload per record

**TLS 1.3 Variations Observed:**
- Packets 1358, 1369-1371: Client Hello with key share
- QUIC protocol using TLS 1.3
- Fewer round trips (1-RTT vs 2-RTT)
- Key exchange in Client Hello
- No separate Change Cipher Spec
- Faster handshake overall

**Certificate Validation:**
1. Signature check: Verified by trusted CA public key
2. Expiration check: Current date within validity period
3. Domain matching: CN/SAN matches requested domain
4. Chain of trust: Intermediate → Root in trusted store

**Session Resumption:**
- Abbreviated handshakes observed (Packets 1355, 1369)
- Using session ID or ticket
- Faster reconnection, reuses encryption key

**Security Properties:**
✓ Confidentiality: All data encrypted after handshake
✓ Integrity: MAC protects against tampering
✓ Authentication: Server proven through certificate chain
✓ Forward Secrecy: ECDHE prevents old sessions compromise

**Security Observations:**
✓ Modern encryption standards (TLS 1.2 and 1.3)
✓ AES-GCM (authenticated encryption)
✓ ECDHE key exchange (forward secrecy)
✓ All servers provided valid certificates
✓ No certificate errors in capture
✓ No downgrade attacks (no SSL 3.0 or TLS 1.0)
✓ All application data encrypted

---

## ARP Protocol Analysis
**Fundamentals:** Layer 3 | Maps IP to MAC addresses | Broadcast on local segment

**How ARP Works:**
Request (broadcast): Device A asks "Who has IP 10.147.191.38?"
Reply (unicast): Device responds "I have that IP, my MAC is 86:f6:79:2f:14:cd"

**Captured ARP Traffic:**

Request Packet 920:
- Source MAC: 86:f6:79:2f:14:cd (gateway)
- Destination MAC: FF:FF:FF:FF:FF:FF (broadcast)
- Source IP: 10.147.191.38
- Target IP: 10.147.191.38 (gratuitous ARP)
- Opcode: Request | Payload: 42 bytes

Reply Packet 921:
- Source MAC: 86:f6:79:2f:14:cd
- Destination MAC: Broadcast
- Response: "10.147.191.38 is at 40:9f:38:7c:65:2e"
- Opcode: Reply | Payload: 42 bytes

**Gratuitous ARP Detected:**
Packets: 920, 6024, 6025, 10063, 10064, 11420, 11421
Purpose: Device announces itself, updates neighbor caches, detects conflicts
Result: No conflicting responses detected

**ARP Packet Structure:**
Hardware Type: 0x0001 (Ethernet)
Protocol Type: 0x0800 (Ipv4)
HW/Protocol Address Lengths: 6/4 bytes
Operation: 1 (Request) or 2 (Reply)
Sender Hardware Address: 6 bytes MAC
Sender Protocol Address: 4 bytes IP
Target Hardware Address: 6 bytes MAC
Target Protocol Address: 4 bytes IP

**IP-to-MAC Mapping Consistency:**
IP: 10.147.191.38 (gateway)
MAC: 86:f6:79:2f:14:cd (AzureWaveTec)
Status: Stable throughout entire 2-3 minute capture

**Cache Behavior:**
- First ARP request for IP → cache miss
- ARP reply received → entry stored
- Subsequent packets use cache (no additional lookups)
- Timeout: Typically 300-600 seconds
- Result: Reduced network traffic

**Security Assessment:**
✓ No ARP spoofing: Same IP always resolves to same MAC
✓ No conflicting responses (prevents MITM at Layer 2)
✓ No ARP scanning: No sequential IP requests
✓ No ARP storms: Regular intervals, no flooding
✓ Broadcast properly handled: No broadcast storms

---

## Protocol Interaction Flow
**User Browsing HTTPS Website (Complete Stack):**

1. **DNS (Layer 7)** → Browser sends query for www.example.com
   - UDP port 53
   - Result: Destination IP obtained (93.184.216.34)

2. **ARP (Layer 3)** → Browser learns gateway MAC
   - Query: Who has gateway IP?
   - Result: MAC address obtained

3. **TCP (Layer 4)** → Three-way handshake
   - SYN → SYN-ACK → ACK
   - Result: Reliable connection channel established

4. **TLS (Layer 6)** → Encryption handshake
   - Certificate exchange and key agreement
   - Result: Session encryption key derived

5. **HTTP (Layer 7)** → Encrypted communication
   - All data encrypted with AES
   - Result: Secure page delivery

**Data Encapsulation Stack:**
HTTP Data
  ↓ (TLS header)
TLS Record
  ↓ (TCP header: ports, sequence, checksum)
TCP Segment
  ↓ (Ipv6 header: source/dest IP, hop limit)
Ipv6 Packet
  ↓ (Ethernet header: source/dest MAC, type)
Ethernet Frame
  ↓
Physical Bits

---

## Packet Statistics
| Protocol | Count | Percentage | Purpose |
|----------|-------|-----------|---------|
| DNS | 96 | 0.8% | Domain resolution |
| TCP | 443 | 3.7% | Connection management |
| TLS | 265 | 2.2% | Encryption/authentication |
| ARP | 42 | 0.4% | IP-to-MAC resolution |
| Other | 10,990 | 92.9% | UDP, ICMPv6, QUIC, etc. |
| **Total** | **11,836** | **100%** | Complete communication |

Other category includes: Ethernet frames, Ipv6 packets, QUIC, ICMPv6, broadcast/multicast, fragmented packets.

---

## Network Health Assessment

**Positive Indicators:**
✓ All DNS queries successful (no NXDOMAIN/SERVFAIL)
✓ All TCP handshakes complete (no connection failures)
✓ Zero packet retransmissions (no packet loss)
✓ No out-of-order delivery
✓ TLS handshakes successful (encryption working)
✓ No certificate errors (valid certificates)
✓ ARP resolution stable and consistent
✓ No network anomalies

**Performance Metrics:**
- TCP handshake: 300 microseconds (excellent)
- TLS handshake: 42 milliseconds (normal for full handshake)
- ARP resolution: Immediate
- Round trip time: 200ms (typical internet)
- Window sizes: Stable (no congestion)
- Throughput: Healthy (full-size segments)

---

## Conclusion
Comprehensive analysis of 11,836 packets demonstrates proper operation across all four analyzed protocols (DNS, TCP, TLS, ARP) across multiple OSI layers. DNS reliably resolves domains. TCP establishes reliable connections with proper sequence/acknowledgment. TLS encrypts all sensitive data with modern cipher suites. ARP reliably maps IP to MAC addresses. No anomalies, errors, or security issues detected. Network operating with modern security practices implemented throughout.

**Analysis Completed:** June 7, 2026
**Total Capture Time:** 2-3 minutes
**Packets Analyzed:** 11,836
**Scope:** 4 major protocols
**Capture File:** packets.pcapng
