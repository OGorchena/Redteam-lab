# Wireshark Lab: Capturing and Analyzing an HTTP GET Request

## Overview

This lab demonstrates how to capture and analyze **HTTP traffic** using Wireshark.  
The objective is to observe how a browser communicates with a web server and to inspect the **GET request**, **HTTP headers**, and **User-Agent** field.

This exercise is useful for understanding:

- how application-layer protocols appear on the network
- how browsers identify themselves to servers
- how unencrypted HTTP traffic can be inspected by network observers

From a cybersecurity perspective, this skill is foundational for both:

- **Blue Team** — traffic analysis, incident response, detection engineering
- **Red Team** — understanding how tools and payloads appear on the network

---

# Lab Objectives

During this lab you will:

1. Capture live network traffic with **Wireshark**
2. Generate HTTP traffic by visiting a website
3. Identify an **HTTP GET request**
4. Inspect **HTTP headers**
5. Locate the **User-Agent string**
6. Analyze the packet across **multiple network layers**

---

# Tools Used

- **Wireshark**
- Web browser (Firefox used in this lab)
- Internet connection

---

# Test Website

For this lab we use a **plain HTTP website**:

```
http://neverssl.com
```

This site intentionally **does not use HTTPS**, allowing us to see HTTP data in plaintext.

---

# Network Environment

Example captured connection:

| Field | Value |
|-----|-----|
| Client IP | `192.168.50.186` |
| Server IP | `34.223.124.45` |
| Protocol | HTTP |
| Destination Port | `80` |

---

# Step 1 — Start Wireshark

1. Open **Wireshark**
2. Select your active network interface

Common interfaces:

- `eth0`
- `wlan0`
- `Wi-Fi`
- `Ethernet`

Start packet capture.

---

# Step 2 — Generate HTTP Traffic

Open your browser and navigate to:

```
http://neverssl.com
```

This will generate:

- DNS requests
- TCP connection
- HTTP requests
- HTTP responses

---

# Step 3 — Filter HTTP Traffic

To reduce noise, apply the following display filter in Wireshark:

```
http
```

This will show only HTTP packets.

Example results:

```
GET / HTTP/1.1
HTTP/1.1 200 OK
HTTP/1.1 301 Moved Permanently
GET /favicon.ico
```

---

# Step 4 — Identify the GET Request

Find a packet containing:

```
GET / HTTP/1.1
```

Example:

| Source | Destination | Protocol | Info |
|------|------|------|------|
| 192.168.50.186 | 34.223.124.45 | HTTP | GET / HTTP/1.1 |

This packet represents the browser requesting the main webpage.

---

# Step 5 — Inspect HTTP Headers

Expanding the **Hypertext Transfer Protocol** section reveals the request headers.

Example captured request:

```
GET / HTTP/1.1
Host: uniquewholeoldstars.neverssl.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Priority: u=4
Pragma: no-cache
Cache-Control: no-cache
```

---

# Important Headers Explained

## Host

```
Host: uniquewholeoldstars.neverssl.com
```

Indicates the **domain name** requested by the browser.

Modern servers often host many domains on a single IP address, so this header tells the server which site the client wants.

---

## User-Agent

```
Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
```

Identifies the client software.

Information revealed:

- Browser → Firefox
- OS → Linux
- Architecture → x86_64

Security relevance:

- defenders use it for **client fingerprinting**
- attackers sometimes **modify or spoof it**
- automated tools often reveal themselves via unusual User-Agent strings

---

## Accept

```
Accept: */*
```

The client accepts any content type.

---

## Accept-Encoding

```
gzip, deflate
```

The browser supports compressed responses.

This explains why the server response includes:

```
Content-Encoding: gzip
```

---

## Connection

```
Connection: keep-alive
```

The client asks the server to **keep the TCP connection open** for additional requests.

---

# Step 6 — Analyze the TCP Layer

Example TCP information from the packet:

```
Source Port: 51904
Destination Port: 80
Flags: PSH, ACK
TCP Segment Length: 302 bytes
```

Explanation:

| Field | Meaning |
|-----|-----|
| Source Port | Ephemeral client port |
| Destination Port | HTTP service port |
| PSH flag | Data should be delivered immediately |
| ACK flag | Acknowledges previous packets |

The HTTP request is carried as **TCP payload**.

---

# Step 7 — Follow the TCP Stream

Wireshark allows reconstruction of the full client-server conversation.

Right click the packet:

```
Follow → TCP Stream
```

Example output:

### Client Request

```
GET / HTTP/1.1
Host: uniquewholeoldstars.neverssl.com
User-Agent: Mozilla/5.0 ...
```

### Server Response

```
HTTP/1.1 200 OK
Server: Apache/2.4.62
Content-Type: text/html
Content-Length: 1900
Content-Encoding: gzip
```

This shows the **complete HTTP exchange**.

---

# Packet Structure Explained

The captured packet contains several protocol layers.

### Ethernet Layer

Contains MAC addresses used on the local network.

### IP Layer

Contains source and destination IP addresses.

Example:

```
Source IP: 192.168.50.186
Destination IP: 34.223.124.45
```

This allows the packet to travel across networks.

---

### TCP Layer

Provides reliable data transport between client and server.

Important functions:

- connection management
- packet ordering
- retransmission
- congestion control

---

### HTTP Layer

The application layer containing the **actual request**.

Example:

```
GET / HTTP/1.1
Host: uniquewholeoldstars.neverssl.com
User-Agent: Mozilla/5.0 ...
```

---

# Screenshots

The following screenshots were captured during the lab.

### HTTP Packet List

Shows GET request and server responses.

### HTTP Headers

Displays parsed HTTP request headers.

### TCP Layer Information

Shows TCP ports, flags, and sequence numbers.

### Follow TCP Stream

Reconstructed HTTP conversation between client and server.

---

# Observations

When the website was opened, the browser performed the following sequence:

1. DNS resolution for the domain name
2. TCP connection establishment
3. HTTP GET request sent to the server
4. Server response with `HTTP/1.1 200 OK`
5. Additional requests for page resources (such as `favicon.ico`)

Because the website used **HTTP instead of HTTPS**, the request headers and content were visible in plaintext.

---

# Security Implications

HTTP traffic can be inspected by:

- network administrators
- attackers on the same network
- compromised routers
- malicious proxies

Sensitive information transmitted over HTTP may therefore be exposed.

This is why modern websites use **HTTPS (TLS encryption)**.

---

# Conclusion

In this lab we captured and analyzed HTTP traffic using Wireshark.  
We successfully identified an HTTP GET request, inspected its headers, extracted the User-Agent string, and examined how the request travels through the network stack.

The lab demonstrates how plaintext protocols can be analyzed at the packet level and highlights the importance of encrypted communication protocols such as HTTPS.

---

# Useful Wireshark Filters

Show HTTP packets:

```
http
```

Show traffic to port 80:

```
tcp.port == 80
```

Show DNS traffic:

```
dns
```

Show packets for a specific stream:

```
tcp.stream eq X
```

---
