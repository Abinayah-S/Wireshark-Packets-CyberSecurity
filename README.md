# Wireshark-Packets-CyberSecurity
## Objective
Capture and analyze live network traffic to identify basic protocols and traffic types using Wireshark.

## Tools Used
- Wireshark 4.0 
- Linux Mint XFCE

## Methodology
1. Launched Wireshark and selected active network interface
2. Started packet capture
3. Generated network traffic by browsing websites and general system activity
4. Captured packets for approximately 1-2 minutes
5. Stopped capture and analyzed traffic patterns
6. Filtered by protocol type (DNS, TCP, TLS, ARP)
7. Exported capture as .pcap file

## Protocols Identified

### 1. DNS (Domain Name System)
- Protocol Type: Application Layer (Layer 7)
- Function: Hostname to IP address resolution
- Captured Queries: brave.com, www.imdb.com, search.brave.com, and others
- Port Used: UDP Port 53
- Key Observation: Standard queries and responses visible in packet data
- Example: Query for brave.com resolved to multiple IP addresses

### 2. TCP (Transmission Control Protocol)
- Protocol Type: Transport Layer (Layer 4)
- Function: Connection-oriented, reliable data transmission
- Connections Observed: Multiple TCP streams with [SYN], [ACK], [RST] flags
- Port Used: Port 443 (HTTPS traffic), Port 36448 and other ephemeral ports
- Key Observation: Handshake sequences visible with sequence number progression
- Window sizes and MSS (Maximum Segment Size) parameters captured

### 3. TLS (Transport Layer Security)
- Protocol Type: Cryptographic Protocol (Layer 6)
- Function: Encryption of application data over TCP
- Version Detected: TLSv1.2, TLSv1.3
- Cipher Specs: Change Cipher Spec, Encrypted Handshake Messages, Application Data
- Key Observation: Cannot decrypt encrypted content (as expected), but protocol structure visible
- Server Details: Encrypted communication between client and remote servers

### 4. ARP (Address Resolution Protocol)
- Protocol Type: Network Layer (Layer 3)
- Function: IP address to MAC address mapping
- Traffic Pattern: ARP who-has queries and ARP tell responses
- Example: Query for 10.147.191.38 mapped to MAC 8e:92:bc:24:46:56
- Key Observation: Local network device discovery and address resolution

## Key Findings

**Total Packets Captured:** 1000+ packets  
**Capture Duration:** Approximately 1-2 minutes  
**Active Protocols:** 4 major protocols identified

**Traffic Distribution:**
- DNS queries dominated early capture (resolution of multiple domains)
- TCP streams represent web browsing (HTTPS connections)
- TLS encryption protects all application data
- ARP traffic occurs during address resolution for gateway communication

**Notable Observations:**
1. Heavy DNS activity indicates frequent web browsing and content fetching
2. Most HTTP traffic uses port 443 (HTTPS), showing security-conscious traffic
3. TLS 1.2 and 1.3 present, with encrypted handshakes observable but not decryptable
4. ARP requests resolve both local and non-local addresses
5. No suspicious or anomalous patterns detected in capture

## Packet Analysis Details

### DNS Packet Example
- Source: 2401:4900:caad:4a4b
- Destination: 2401:4900:caad:4a4b
- Query: AAAA record lookup for brave.com
- Response: Multiple AAAA records (IPv6 addresses) returned

### TCP Packet Example
- Source: 2600:9000:2579:e00
- Destination: 2401:4900:caad:4a4b
- Flags: [SYN], [ACK], [RST]
- Sequence Numbers: Progressive increments indicating reliable delivery
- Window Size: 70656 bytes

### TLS Handshake Example
- Client Hello observed
- Server Hello with cipher suite negotiation
- Certificate exchange (encrypted)
- Change Cipher Spec message
- Application Data (fully encrypted)

### ARP Resolution Example
- "Who has 10.147.191.38?" query
- Response: MAC address 8e:92:bc:24:46:56 at that IP
- Protocol: Confirms ARP is functional for local network discovery

## Practical Implications

**Troubleshooting:** Packet capture reveals connection bottlenecks, packet loss, and latency issues  
**Security:** Encrypted traffic (TLS) prevents eavesdropping, but metadata (IPs, ports, timing) remains visible  
**Protocol Understanding:** Actual packet structure visible in hex dump helps understand OSI model layers  

## Files Included
- `packet_capture.pcap`: Raw packet capture file
- `analysis_report.md`: Detailed protocol analysis
- .png images given are filtered views of TCP,DNS,TLS and ARP traffic
- This README with complete methodology and findings

## Conclusion
Successfully captured and analyzed 1000+ network packets, identifying 4 distinct protocols (DNS, TCP, TLS, ARP) operating at different OSI layers. Packet analysis demonstrates understanding of network communication fundamentals and practical use of Wireshark for network diagnostics.
