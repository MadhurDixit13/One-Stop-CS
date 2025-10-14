# TCP and UDP

TCP (Transmission Control Protocol) and UDP (User Datagram Protocol) are the two primary transport layer protocols in the Internet Protocol Suite, responsible for delivering data between applications over a network.

---

## What Problem Do They Solve?

Transport layer protocols solve:
- **Application-to-Application Communication**: Enable processes on different hosts to exchange data
- **Multiplexing**: Multiple applications sharing the same network connection (via port numbers)
- **Data Delivery**: Ensure data arrives correctly (TCP) or quickly (UDP)
- **Error Handling**: Detect and recover from network issues (TCP) or leave it to the application (UDP)

---

## TCP (Transmission Control Protocol)

### What It Is
TCP is a **connection-oriented**, **reliable**, **ordered** transport protocol that guarantees data delivery.

### Key Features
- **Connection-Oriented**: Three-way handshake (SYN, SYN-ACK, ACK) to establish connection
- **Reliable Delivery**: Acknowledgments, retransmissions, timeouts
- **Ordered Delivery**: Sequence numbers ensure data arrives in order
- **Flow Control**: Sliding window to prevent sender from overwhelming receiver
- **Congestion Control**: Adapts to network conditions (slow start, congestion avoidance)
- **Error Detection**: Checksums to detect corrupted packets

### When to Use TCP
- **Web Browsing**: HTTP/HTTPS
- **Email**: SMTP, IMAP, POP3
- **File Transfer**: FTP, SFTP, SCP
- **Remote Access**: SSH, Telnet
- **Database Connections**: MySQL, PostgreSQL
- **APIs**: REST APIs over HTTP/HTTPS
- **Any scenario where reliability is critical**

### TCP Header (20-60 bytes)
```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |       |C|E|U|A|P|R|S|F|                               |
| Offset| Rsvd  |W|C|R|C|S|S|Y|I|            Window             |
|       |       |R|E|G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### TCP Connection Lifecycle
1. **Establishment**: Three-way handshake (SYN → SYN-ACK → ACK)
2. **Data Transfer**: Send/receive data with acknowledgments
3. **Termination**: Four-way handshake (FIN → ACK → FIN → ACK)

---

## UDP (User Datagram Protocol)

### What It Is
UDP is a **connectionless**, **unreliable**, **unordered** transport protocol optimized for speed and low overhead.

### Key Features
- **Connectionless**: No handshake; send data immediately
- **Unreliable**: No acknowledgments, retransmissions, or guarantees
- **Unordered**: Packets may arrive out of order or not at all
- **Low Overhead**: 8-byte header (vs. 20+ bytes for TCP)
- **Multicast/Broadcast**: Supports one-to-many communication
- **Faster**: No connection setup or reliability mechanisms

### When to Use UDP
- **Streaming**: Video, audio (Zoom, YouTube Live, Netflix QUIC)
- **Gaming**: Real-time multiplayer games (low latency critical)
- **VoIP**: Voice calls (Skype, WhatsApp calls)
- **DNS**: Fast lookups (53/udp)
- **IoT**: Sensor data, telemetry (MQTT over UDP)
- **Monitoring**: SNMP, syslog
- **Any scenario where speed > reliability**

### UDP Header (8 bytes)
```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

---

## TCP vs. UDP Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | ✅ Connection-oriented | ❌ Connectionless |
| **Reliability** | ✅ Guaranteed delivery | ❌ No guarantees |
| **Ordering** | ✅ In-order delivery | ❌ May arrive out of order |
| **Speed** | 🐢 Slower (overhead) | 🚀 Faster (minimal overhead) |
| **Header Size** | 20-60 bytes | 8 bytes |
| **Flow Control** | ✅ Yes | ❌ No |
| **Congestion Control** | ✅ Yes | ❌ No |
| **Error Checking** | ✅ Extensive | ⚠️ Basic checksum |
| **Use Case** | File transfer, web, email | Streaming, gaming, VoIP |
| **Example Protocols** | HTTP, FTP, SSH | DNS, DHCP, TFTP, QUIC |

---

## Port Numbers

Both TCP and UDP use **port numbers** (0-65535) to identify applications:
- **Well-Known Ports (0-1023)**: HTTP (80), HTTPS (443), SSH (22), DNS (53), FTP (21)
- **Registered Ports (1024-49151)**: MySQL (3306), PostgreSQL (5432), Redis (6379)
- **Dynamic/Private Ports (49152-65535)**: Ephemeral ports assigned by OS

---

## Modern Protocols Building on TCP/UDP

### QUIC (Quick UDP Internet Connections)
- Built on UDP but adds reliability, encryption, and multiplexing
- Used by HTTP/3 for faster web browsing
- **Advantages**: Lower latency, better performance over lossy networks

### HTTP/2 vs. HTTP/3
- **HTTP/2**: Uses TCP (suffers from head-of-line blocking)
- **HTTP/3**: Uses QUIC over UDP (solves head-of-line blocking)

### WebRTC
- Peer-to-peer communication (video calls, file sharing)
- Uses UDP for media streams, TCP for signaling

---

## Best Practices

### When to Choose TCP
- Reliability is critical (data loss unacceptable)
- Order matters (file transfers, database queries)
- Connection state is useful (sessions, authentication)

### When to Choose UDP
- Speed and low latency are critical
- Occasional packet loss is acceptable
- Real-time applications (gaming, live video, VoIP)
- Multicast/broadcast required

### Security Considerations
- **TCP**: Vulnerable to SYN flood attacks, session hijacking
- **UDP**: Vulnerable to amplification attacks (DNS, NTP)
- **Mitigation**: Firewalls, rate limiting, encryption (TLS for TCP, DTLS for UDP)

---

## Further Reading

- [RFC 793 - TCP](https://datatracker.ietf.org/doc/html/rfc793)
- [RFC 768 - UDP](https://datatracker.ietf.org/doc/html/rfc768)
- [Computer Networking: A Top-Down Approach](https://gaia.cs.umass.edu/kurose_ross/index.php)
- [TCP Congestion Control](https://en.wikipedia.org/wiki/TCP_congestion_control)
- [QUIC Protocol](https://www.chromium.org/quic/)
- [HTTP/3 Explained](https://http3-explained.haxx.se/)
- [Cloudflare: TCP vs UDP](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/)

