# Case 3: Packet Primer - PicoCTF Write-up

## Overview

This write-up explains how I solved the **Packet Primer** challenge from **PicoCTF** using **Wireshark**.

The objective of this challenge is to analyze a packet capture (PCAP) file, understand the TCP communication process, inspect the transmitted data, and recover the hidden PicoCTF flag.

Unlike many forensic challenges that require advanced analysis, this challenge introduces the fundamentals of network packet analysis and demonstrates how useful information can be extracted directly from captured network traffic.

---

# Challenge Information

| Category | Value |
|----------|-------|
| Platform | PicoCTF |
| Challenge Name | Packet Primer |
| Category | Forensics |
| Tool Used | Wireshark |
| File Provided | `network-dump.flag.pcap` |

---

# Objective

Analyze the provided **PCAP (Packet Capture)** file and identify the hidden PicoCTF flag.

---

# What is a PCAP File?

A **PCAP (Packet Capture)** file is a file that stores captured network traffic. It records every packet transmitted between devices during a network communication.

A PCAP file may contain:

- TCP packets
- UDP packets
- HTTP traffic
- DNS queries
- ARP packets
- ICMP packets
- Many other network protocols

PCAP files are widely used in:

- Digital Forensics
- Incident Response
- Malware Analysis
- Threat Hunting
- Network Troubleshooting
- Security Monitoring

Think of a PCAP file as a recording of everything that happened on a network during a particular period of time.

---

# Step 1: Download the Challenge File

Open the **Packet Primer** challenge on PicoCTF and download the provided packet capture file.

```
network-dump.flag.pcap
```

This PCAP file contains the captured network communication that needs to be analyzed.

---

# Step 2: Open the PCAP File in Wireshark

Launch **Wireshark** and open the downloaded file.

```
File → Open
```

Select:

```
network-dump.flag.pcap
```

Once opened, Wireshark displays all the captured packets contained in the file.

> **Insert Screenshot 1 Here**
>
> *Initial Packet View in Wireshark*

---

# Initial Packet Analysis

After opening the PCAP file, we can observe several packets exchanged between two systems.

### Source IP

```
10.0.2.15
```

### Destination IP

```
10.0.2.4
```

The packet sequence is:

```
1. SYN
2. SYN, ACK
3. ACK
4. PSH, ACK
5. ACK
```

This sequence represents a normal TCP communication.

Before sending any actual data, TCP first establishes a reliable connection between the client and the server. This process is known as the **TCP Three-Way Handshake**.

---

# Understanding the TCP Three-Way Handshake

TCP is a **connection-oriented protocol**, meaning both devices must establish a connection before exchanging data.

The connection is established using three packets:

```
SYN → SYN-ACK → ACK
```

This process ensures that both devices are online, synchronized, and ready to communicate.

---

# Packet 1 – SYN

```
10.0.2.15 → 10.0.2.4
[SYN]
```

The first packet is sent from the client (`10.0.2.15`) to the server (`10.0.2.4`).

The **SYN (Synchronize)** flag indicates that the client wants to establish a TCP connection.

At this stage:

- No actual data is transmitted.
- The client is simply requesting to start communication.

You can think of this as someone knocking on a door and asking,

> "Can we start communicating?"

---

# Packet 2 – SYN, ACK

```
10.0.2.4 → 10.0.2.15
[SYN, ACK]
```

The server replies with a **SYN-ACK** packet.

This response has two purposes:

- **SYN** indicates that the server is also ready to establish the connection.
- **ACK (Acknowledgment)** confirms that the server successfully received the client's SYN packet.

In simple terms, the server is saying:

> "Yes, I received your request, and I'm ready to communicate."

---

# Packet 3 – ACK

```
10.0.2.15 → 10.0.2.4
[ACK]
```

The client sends a final **ACK** packet back to the server.

This packet confirms that the client has received the server's SYN-ACK response.

At this point:

- Both systems have successfully exchanged acknowledgments.
- The TCP connection is fully established.
- Data transmission can now begin.

This completes the **TCP Three-Way Handshake**.

---

# Why is the Three-Way Handshake Important?

The TCP Three-Way Handshake ensures that:

- Both devices are reachable.
- Both devices are ready to communicate.
- Sequence numbers are synchronized.
- Reliable communication can begin.

Without this process, TCP cannot guarantee reliable data transmission.

---

# Packet 4 – PSH, ACK

After the TCP connection has been established, the next packet is:

```
PSH, ACK
```

> **Insert Screenshot 2 Here**
>
> *PSH, ACK Packet*

Unlike the previous packets, this packet contains **actual application data**.

---

# What Does PSH Mean?

**PSH (Push)** tells the receiving system to immediately pass the received data to the application instead of waiting for additional packets.

Normally, TCP may temporarily buffer incoming data before delivering it.

With the **PSH** flag set, TCP instructs the receiver:

> "Deliver this data immediately."

---

# What Does ACK Mean Here?

The **ACK** flag acknowledges that all previously received packets have arrived successfully.

Therefore, this packet performs two tasks:

- Acknowledges previous communication.
- Carries actual application data.

---

# Inspecting the Packet Payload

Select the **PSH, ACK** packet in Wireshark.

In the Packet Details pane, expand:

```
Transmission Control Protocol
```

Then expand:

```
Data
```

You will notice that the TCP payload contains **60 bytes** of data.

Wireshark displays the payload in three different formats:

- Packet Details
- Hexadecimal View
- ASCII View

---

# Understanding the Hexadecimal View

Every piece of information transmitted across a network is represented internally as hexadecimal values.

For example:

```
70 69 63 6f ...
```

Each pair of hexadecimal digits represents one byte.

Although these values may appear unreadable, they actually represent characters that can be converted into text.

---

# Understanding the ASCII View

On the right side of Wireshark, the same hexadecimal bytes are automatically converted into their ASCII representation.

Instead of seeing hexadecimal values, we can now read the transmitted text.

This makes it much easier to identify readable information hidden inside packet payloads.

> **Insert Screenshot 3 Here**
>
> *Hexadecimal and ASCII View*

---

# Finding the Flag

While examining the TCP payload in the ASCII view, the PicoCTF flag becomes visible.

The transmitted payload contains the challenge flag directly within the packet data.

Since Wireshark automatically converts printable hexadecimal values into ASCII characters, no additional decoding is required.

This packet contains the required PicoCTF flag, which completes the challenge.

---

# Key Learning

Through this challenge, I learned:

- What a PCAP file is.
- How to open and analyze PCAP files using Wireshark.
- How TCP establishes connections using the Three-Way Handshake.
- The purpose of SYN, SYN-ACK, ACK, and PSH packets.
- How to inspect TCP packet payloads.
- How hexadecimal data is represented.
- How ASCII conversion helps reveal readable information.
- How packet analysis can uncover hidden data such as flags.

---

# Conclusion

The **Packet Primer** challenge provides an excellent introduction to network forensics and TCP packet analysis.

By opening the PCAP file in Wireshark, understanding the TCP Three-Way Handshake, analyzing the captured packets, and inspecting the TCP payload, the hidden PicoCTF flag can be successfully recovered.

Although simple, this challenge teaches fundamental networking concepts that are essential for Digital Forensics, Incident Response, Threat Hunting, and Security Operations Center (SOC) investigations.

---
