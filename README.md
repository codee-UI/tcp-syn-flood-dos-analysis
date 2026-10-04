# TCP SYN Flood DoS Attack Analysis

## Repository Description

Wireshark-based analysis of a TCP SYN flood DoS attack affecting an HTTPS web server, including packet analysis, service impact, incident response, and mitigation.

## Overview

This repository documents the analysis of a simulated cybersecurity incident involving a web server affected by a **TCP SYN flood Denial-of-Service (DoS) attack**.

The incident was investigated using packet-capture data to identify abnormal TCP behavior, determine how the attack affected legitimate users, and document appropriate mitigation measures.

The analysis focuses on:

- TCP three-way handshake behavior
- SYN flood attack patterns
- Web server availability
- Wireshark packet analysis
- HTTP and TCP failure indicators
- Firewall mitigation
- Incident-response recommendations

---

## Scenario

A travel agency uses a company website to advertise vacation packages and promotions. Employees regularly access the website to search for offers for customers.

An automated monitoring system generated an alert indicating a problem with the web server.

When the website was tested, the browser returned a:

`Connection timeout`

Packet-capture analysis showed an unusually large number of TCP SYN requests coming from an unfamiliar IP address.

The web server became overwhelmed by the incoming connection requests and gradually lost the ability to respond to legitimate users.

---

## Incident Classification

**Attack Type:** TCP SYN Flood  
**Attack Category:** Denial-of-Service (DoS)  
**Target Service:** HTTPS  
**Target Port:** TCP 443  
**Target Server:** `192.0.2.1`  
**Observed Attack Source:** `203.0.113.0`  
**Primary Security Impact:** Availability

The supplied traffic analysis identifies the event as a direct DoS SYN flood because the attack traffic originates from one observed source IP address.

---

## Normal TCP Connection

A normal TCP connection uses a three-way handshake.

### Step 1 — SYN

The client requests a connection.

```text
Client -------- SYN --------> Server
```

### Step 2 — SYN-ACK

The server acknowledges the request.

```text
Client <----- SYN-ACK ------- Server
```

### Step 3 — ACK

The client acknowledges the server response.

```text
Client -------- ACK --------> Server
```

The completed TCP handshake is:

```text
Client                         Server
   |                              |
   | ---------- SYN ------------> |
   |                              |
   | <------- SYN-ACK ----------- |
   |                              |
   | ---------- ACK ------------> |
   |                              |
   |      Connection Open         |
```

Once the connection is established, normal web traffic can begin.

---

## Normal Website Traffic

Under normal conditions, an employee establishes a TCP connection with the web server and then requests the webpage.

Example traffic:

```text
Client -> Server
TCP [SYN]

Server -> Client
TCP [SYN, ACK]

Client -> Server
TCP [ACK]

Client -> Server
HTTP GET /sales.html

Server -> Client
HTTP/1.1 200 OK
```

The supplied log shows legitimate users successfully completing the TCP handshake and receiving `HTTP/1.1 200 OK` responses before the attack begins to overwhelm the server.

---

## Attack Behavior

The attack source repeatedly sends TCP SYN packets to the web server.

Observed pattern:

```text
203.0.113.0 -> 192.0.2.1
TCP 54770 -> 443 [SYN]
```

Instead of normal connection behavior, the SYN requests continue at an abnormal rate.

The web server must process these requests and allocate resources for TCP connections.

As the number of requests increases, the server's available resources become exhausted.

---

## SYN Flood Process

```text
Attacker
   |
   | SYN
   | SYN
   | SYN
   | SYN
   | SYN
   v
Web Server
   |
   v
Connection resources consumed
   |
   v
Legitimate requests delayed
   |
   v
Connection failures
   |
   v
Website unavailable
```

A SYN flood abuses the first stage of the TCP connection-establishment process by sending a large number of SYN requests. If the number of requests exceeds available server resources, the server becomes overwhelmed and cannot respond normally.

---

## Packet Analysis Findings

The attack traffic originates from:

`203.0.113.0`

and targets:

`192.0.2.1`

on:

`TCP Port 443`

The logs show repeated SYN packets such as:

```text
54770 -> 443 [SYN]
```

The same source continues sending SYN packets throughout the capture.

Initially, the server can still respond to legitimate traffic. Later, the volume of malicious traffic increases and legitimate communications begin to fail.

---

## Evidence of Service Degradation

The packet capture contains two important failure indicators.

### HTTP 504 Gateway Timeout

```text
HTTP/1.1 504 Gateway Time-out
```

This indicates that a gateway waited for the web server to respond but did not receive a response within the required time.

### TCP RST, ACK

```text
[RST, ACK]
```

These packets indicate failed or reset connection attempts.

As the SYN traffic increases, legitimate users begin receiving timeout errors and TCP reset responses.

---

## Attack Progression

### Early Stage

The server continues handling legitimate users.

A normal employee can:

1. Send SYN
2. Receive SYN-ACK
3. Send ACK
4. Request `/sales.html`
5. Receive `HTTP/1.1 200 OK`

### Degraded Stage

As the attack continues:

- SYN traffic increases
- Server resources become constrained
- Legitimate handshakes begin failing
- Users receive timeout errors
- Reset packets appear

### Final Stage

Eventually, the web server stops responding to legitimate employee traffic.

The later portion of the supplied log contains only attack traffic, showing that the server has effectively become unavailable.

---

## DoS vs DDoS

### DoS

A Denial-of-Service attack generally originates from one source.

```text
Attacker ---> Target Server
```

### DDoS

A Distributed Denial-of-Service attack involves multiple attacking systems.

```text
Attacker 1 \
Attacker 2  \
Attacker 3 ---> Target Server
Attacker 4  /
Attacker 5 /
```

The activity classifies this incident as a direct DoS attack because one attacking IP address is observed.

---

## Impact on the Organization

The attack affects the availability of the travel agency's web service.

### Employee Impact

Employees may experience:

- Slow website responses
- Connection timeouts
- Failed TCP sessions
- Inability to access travel promotions
- Reduced productivity

### Customer Impact

Potential customer effects include:

- Inability to access promotions
- Failed website sessions
- Poor customer experience
- Difficulty viewing vacation packages

### Business Impact

Potential organizational consequences include:

- Website downtime
- Lost sales opportunities
- Reduced employee productivity
- Increased incident-response workload
- Customer dissatisfaction
- Reputational damage

---

## CIA Triad Impact

The primary security property affected is:

**Availability**

```text
Confidentiality  -> No direct evidence of compromise
Integrity        -> No direct evidence of modification
Availability     -> Significantly affected
```

The goal of the attack is to make the web service unavailable to legitimate users.

---

## Immediate Response

The scenario describes two immediate actions.

### 1. Temporarily Take the Server Offline

The server was temporarily removed from service so that it could recover.

### 2. Block the Attacking IP Address

The firewall was configured to block:

`203.0.113.0`

This reduces malicious traffic from the identified source.

---

## Limitation of IP Blocking

Blocking one source IP provides only temporary protection.

An attacker may:

- Change IP addresses
- Spoof source addresses
- Use multiple compromised systems
- Launch future attacks from different networks

Therefore, additional protections are required.

---

## Recommended Mitigations

### SYN Flood Protection

Enable TCP SYN flood protection mechanisms.

### SYN Cookies

Use SYN cookies where appropriate to reduce resource allocation for incomplete connections.

### Rate Limiting

Limit excessive connection attempts from individual sources.

### Firewall Rules

Configure firewall rules to detect and restrict abnormal SYN traffic.

### IDS / IPS

Use intrusion detection or prevention systems to identify SYN flood patterns.

### Traffic Monitoring

Monitor connection rates and establish alerts for abnormal traffic volumes.

### Load Balancing

Distribute traffic across multiple servers where appropriate.

### Upstream DDoS Protection

For public-facing services, upstream filtering or DDoS mitigation services can provide additional protection against larger or distributed attacks.

---

## Incident Flow

```text
Unfamiliar source
203.0.113.0
        |
        | Repeated TCP SYN requests
        v
Firewall
        |
        v
Web Server
192.0.2.1:443
        |
        v
Connection resources consumed
        |
        v
Server performance degrades
        |
        +----------------------+
        |                      |
        v                      v
504 Gateway Timeout        RST / ACK
        |                      |
        +----------+-----------+
                   |
                   v
       Legitimate users unable
           to access website
```

---

## Indicators

| Indicator | Observation |
|---|---|
| Attack type | TCP SYN flood |
| Attack category | DoS |
| Source IP | `203.0.113.0` |
| Target IP | `192.0.2.1` |
| Target port | TCP `443` |
| Protocol | TCP |
| Service | HTTPS |
| Malicious pattern | Repeated SYN requests |
| HTTP symptom | `504 Gateway Time-out` |
| TCP symptom | `RST, ACK` |
| User symptom | Connection timeout |
| CIA impact | Availability |

---

## Network Components Involved

| Component | Role |
|---|---|
| Employee workstation | Legitimate website client |
| Attacking host | Generates SYN flood traffic |
| Firewall | Filters incoming network traffic |
| Web server | Target of the attack |
| Gateway | May return HTTP 504 errors |
| Monitoring system | Detects abnormal conditions |
| Packet analyzer | Used to investigate network traffic |

---

## Skills Demonstrated

This project demonstrates practical understanding of:

- Network traffic analysis
- Wireshark
- TCP/IP
- TCP three-way handshake
- SYN flood detection
- Denial-of-Service analysis
- HTTP status codes
- Packet inspection
- Firewall mitigation
- Incident response
- Network troubleshooting
- Cybersecurity documentation

---

## Tools and Technologies

- Wireshark
- TCP
- HTTP / HTTPS
- TCP Port 443
- Firewall
- Packet Analysis
- Network Monitoring

---

## Key Findings

1. The web server received an abnormal number of TCP SYN requests.
2. The attack traffic originated from `203.0.113.0`.
3. The target server was `192.0.2.1`.
4. TCP port 443 was targeted.
5. Legitimate traffic initially succeeded.
6. The server gradually became overwhelmed.
7. HTTP 504 Gateway Timeout errors appeared.
8. TCP reset packets appeared.
9. Legitimate users eventually could not access the server.
10. The incident is consistent with a direct TCP SYN flood DoS attack.

---

## Conclusion

The network traffic analysis identified a **TCP SYN flood Denial-of-Service attack** targeting the travel agency's HTTPS web server.

The attacker repeatedly sent TCP SYN requests to port 443, consuming server resources and reducing its ability to process legitimate connections.

As the attack progressed, employees experienced connection failures, TCP resets, and HTTP gateway timeout errors.

The firewall block against the observed source provided temporary mitigation, but stronger defensive controls such as SYN flood protection, rate limiting, IDS/IPS monitoring, and traffic filtering are recommended for improved resilience.

---

## Repository Purpose

This repository is maintained as part of a cybersecurity learning and professional portfolio.

It demonstrates the ability to:

- Analyze packet captures
- Identify abnormal TCP behavior
- Recognize network attacks
- Assess service impact
- Interpret TCP and HTTP errors
- Document incident findings
- Recommend defensive controls

---

## Disclaimer

This repository is based on a simulated cybersecurity training scenario and is intended for educational and portfolio purposes.

The IP addresses, organization, network traffic, and attack activity used in this exercise do not represent an actual security incident.

---

## Suggested GitHub Topics

`cybersecurity` `wireshark` `network-security` `tcp` `syn-flood` `dos-attack` `packet-analysis` `incident-response` `network-analysis` `https`






---

## Keywords

`Cybersecurity` `Network Security` `Wireshark` `TCP` `SYN Flood` `DoS` `Denial of Service` `Packet Analysis` `Incident Response` `HTTPS` `Port 443` `Firewall` `IDS` `IPS` `Network Traffic Analysis`

# Interception Tactics — Packet Sniffing, IP Spoofing, On-Path and Smurf Attacks

## Packet Sniffing

**Packet sniffing** is the practice of capturing and inspecting data packets as they travel across a network.

On a private network, packets are normally delivered to the intended destination device. Each device contains a **Network Interface Card (NIC)**, which connects the device to the network.

Under normal operation, the NIC:
1. Receives network traffic
2. Reads destination information
3. Checks whether the packet contains the device's MAC address
4. Accepts matching packets
5. Passes the packet to the operating system or application

## Promiscuous Mode

A NIC can be configured to operate in **promiscuous mode**. In this mode, it accepts network traffic even when packets are not addressed to that device.

This can be useful for legitimate troubleshooting, but malicious actors may abuse it with tools such as **Wireshark** to capture traffic and inspect:

- IP addresses
- MAC addresses
- Network protocols
- Authentication information
- Unencrypted data
- User activity

## Packet Sniffing Flow

```text
Network Traffic
      |
      v
Network Interface Card
      |
      +-------------------------+
      |                         |
 Normal Mode              Promiscuous Mode
      |                         |
Only packets             Captures additional
for device               network traffic
      |                         |
      v                         v
Normal processing       Packet inspection
                              |
                              v
                     Potential information
                         disclosure
```

# IP Spoofing

After capturing traffic, a malicious actor may impersonate an authorized system.

**IP spoofing** involves modifying the source IP information in a packet so it appears to come from another device.

Information gathered through packet sniffing may include:
- IP addresses
- MAC addresses
- Network configuration information

## Firewall Protection Against IP Spoofing

Firewalls can help reduce IP spoofing by rejecting:
- Unauthorized IP packets
- Invalid source addresses
- Unexpected traffic
- Suspicious packet patterns

# On-Path Attack

An **on-path attack** occurs when a malicious actor intercepts communication between two trusted devices or servers.

```text
Device A
   |
   v
Attacker
   |
   v
Device B
```

The attacker may:
- Observe communications
- Capture sensitive information
- Modify transmitted data
- Redirect traffic
- Impersonate one of the communicating systems

An on-path attack may also be called a **meddler-in-the-middle attack**.

## Information at Risk

Intercepted communication may contain:
- Usernames
- Passwords
- Authentication information
- Session information
- Personal information
- Network requests
- DNS queries

## DNS Manipulation in an On-Path Attack

DNS translates domain names into IP addresses.

```text
User
 |
 | Request: example.com
 v
DNS Server
 |
 | Response: 192.0.2.20
 v
User
```

During an on-path attack, a malicious actor may intercept the DNS lookup and attempt to provide a fraudulent DNS response that redirects the user to a malicious IP address.

## Protection Against On-Path Attacks

A major defense is **encrypting data in transit**, for example with **Transport Layer Security (TLS)**.

# Smurf Attack

A **smurf attack** is a network attack involving spoofed IP information and large amounts of ICMP traffic.

The attacker uses the victim's IP address and sends packets to a network broadcast address.

```text
                    Attacker
                       |
                       | ICMP request
                       | Spoofed source IP
                       v
               Broadcast Address
                /      |      \
               /       |       \
              v        v        v
          Device 1  Device 2  Device 3
              \        |        /
               \       |       /
                \      |      /
                 v     v     v
                    Victim
```

Multiple systems may respond to the spoofed address, overwhelming the target.

## ICMP and Smurf Attacks

**Internet Control Message Protocol (ICMP)** is commonly used for diagnostics such as `ping`.

During a smurf attack, excessive ICMP responses can cause:
- Network congestion
- Resource exhaustion
- Slow performance
- Service interruption
- Server unavailability

## Protecting Against Smurf Attacks

Useful controls include:
- Advanced firewalls
- Next-generation firewalls
- Traffic monitoring
- Network anomaly detection
- ICMP filtering
- Broadcast traffic controls

# Denial-of-Service Attack

A **Denial-of-Service (DoS) attack** attempts to prevent a system from performing legitimate activities or responding to legitimate traffic.

```text
Attacker
   |
   | Large volume of traffic
   v
Target Server
   |
   v
Resources consumed
   |
   v
Performance degradation
   |
   v
Legitimate users denied access
```

A DoS attack may cause:
- Slow response times
- Connection failures
- Service disruption
- Website unavailability
- Server resource exhaustion
- Network congestion

The primary security property affected is **Availability**.

# Relationship Between the Attacks

```text
Packet Sniffing
      |
      v
Information Gathering
      |
      v
IP / MAC Information Obtained
      |
      v
IP Spoofing
      |
      +-------------------+
      |                   |
      v                   v
On-Path Attack       Smurf Attack
      |                   |
      v                   v
Data interception    ICMP traffic flood
      |                   |
      v                   v
Credential / Data       DoS
Exposure             Service Disruption
```

# Comparison Table

| Attack / Technique | Main Purpose | Typical Method | Primary Impact |
|---|---|---|---|
| **Packet Sniffing** | Capture network traffic | Packet analysis tools | Information exposure |
| **Active Packet Sniffing** | Capture or manipulate traffic | Interaction with network traffic | Confidentiality / Integrity |
| **Passive Packet Sniffing** | Observe traffic | Monitoring network packets | Confidentiality |
| **IP Spoofing** | Impersonate another system | Fake source IP address | Authentication / Trust |
| **On-Path Attack** | Intercept trusted communication | Position attacker between systems | Confidentiality / Integrity |
| **Smurf Attack** | Overwhelm target with traffic | Spoofed IP + ICMP broadcast | Availability |
| **DoS Attack** | Make a service unavailable | Resource or traffic exhaustion | Availability |

# Security Controls

## Encryption
Use:
- TLS
- HTTPS
- VPN technologies

## Firewalls
Firewalls can:
- Block suspicious traffic
- Restrict unauthorized IP addresses
- Filter packets
- Apply ingress and egress rules
- Detect abnormal traffic patterns

## Network Monitoring
Monitor:
- Unusual traffic volumes
- Unexpected source addresses
- Large numbers of ICMP packets
- Abnormal broadcast traffic
- Suspicious connection patterns

## Network Segmentation

```text
Internet
   |
Firewall
   |
   +-------------+
   |             |
User Network   Server Network
   |             |
   +-------------+
          |
      Restricted
      Communication
```

## Defense in Depth

Use multiple security layers:

```text
Internet
   |
   v
Firewall
   |
   v
IDS / IPS
   |
   v
Network Segmentation
   |
   v
Encryption
   |
   v
Endpoint Security
   |
   v
Monitoring & Logging
```

# Key Takeaways

1. Packet sniffing captures and inspects network traffic.
2. Promiscuous mode allows a NIC to accept traffic not specifically addressed to it.
3. Packet sniffing may expose IP addresses, MAC addresses, and sensitive information.
4. IP spoofing allows an attacker to impersonate another system's IP address.
5. On-path attacks intercept communication between trusted systems.
6. TLS encryption is an important defense against interception of data in transit.
7. Smurf attacks combine IP spoofing, ICMP traffic, and DoS techniques.
8. DoS attacks attempt to make services unavailable.
9. Firewalls and network monitoring can help identify suspicious traffic.
10. Defense in depth uses multiple security controls rather than relying on a single protection mechanism.

# Skills Demonstrated

- Packet sniffing
- Network traffic analysis
- Wireshark
- Network Interface Cards
- Promiscuous mode
- IP spoofing
- On-path attacks
- DNS interception
- ICMP
- Smurf attacks
- Denial-of-Service attacks
- Firewall security
- Encryption
- TLS
- Network monitoring
- Defense in depth

## Keywords

`Cybersecurity` `Network Security` `Packet Sniffing` `Wireshark` `IP Spoofing` `On-Path Attack` `Smurf Attack` `DoS` `ICMP` `DNS` `TLS` `Firewall` `Promiscuous Mode` `Network Monitoring` `Defense in Depth`




