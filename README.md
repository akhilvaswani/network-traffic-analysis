# Network Traffic Analysis with Wireshark

In this project, I set up a cloud environment to capture and analyze network traffic using Wireshark. I walked through different types of traffic - ICMP, SSH, DNS, HTTP, and RDP - and used filters to isolate and understand each protocol.

This is a foundational skill for IT and cybersecurity - being able to look at raw network traffic and understand what's happening on the wire.

## Environments and Technologies Used

- Microsoft Azure (Virtual Machines, Virtual Networks)
- Wireshark (network protocol analyzer)
- Command line tools (ping, nslookup, ipconfig, ssh)
- Windows 10 Pro and Ubuntu Server

## Operating Systems Used

- Windows 10 (running Wireshark)
- Ubuntu Server 22.04 (target machine for SSH traffic)

## Steps

### 1 - Set Up the Environment

I created two VMs in the same Azure Virtual Network:

- **VM1 (Windows 10)** - This is where I installed Wireshark and did the capturing
- **VM2 (Ubuntu Server)** - This was the target machine I communicated with

Both VMs were on the same subnet so I could capture traffic between them without any routing complications.

### 2 - Install Wireshark

After connecting to VM1 via Remote Desktop, I downloaded and installed Wireshark from [wireshark.org](https://www.wireshark.org). During installation I made sure to include the Npcap packet capture driver - Wireshark needs this to actually see the network traffic.

### 3 - Capture ICMP Traffic (Ping)

I started a capture in Wireshark and then opened a Command Prompt to ping VM2's private IP:

```
ping 10.0.0.5
```

In Wireshark, I filtered for ICMP traffic only:

```
icmp
```

I could see the Echo Request going from VM1 to VM2, and the Echo Reply coming back. Each packet showed the source IP, destination IP, and the payload. I also pinged a public address (google.com) to see the difference in response times - local pings were under 1ms while external pings were around 10-15ms.

### 4 - Capture SSH Traffic

Next I SSH'd into VM2 from VM1 to generate some encrypted traffic:

```
ssh adminuser@10.0.0.5
```

In Wireshark, I filtered for SSH:

```
tcp.port == 22
```

The interesting thing here is that you can see the TCP handshake and the SSH key exchange, but the actual data is encrypted. All Wireshark shows is "Encrypted packet" for the payload - which is exactly what you'd expect from SSH.

### 5 - Capture DNS Traffic

I used `nslookup` to generate DNS queries:

```
nslookup google.com
nslookup amazon.com
```

With the DNS filter in Wireshark:

```
dns
```

I could see the full DNS resolution process - the query going out to the DNS server asking "what's the IP for google.com?" and the response coming back with the A record. Each query-response pair was clearly visible.

### 6 - Capture HTTP/HTTPS Traffic

I opened a web browser on VM1 and visited a few websites. In Wireshark:

```
tcp.port == 80 || tcp.port == 443
```

Most modern traffic is HTTPS (port 443), so like SSH, the payload is encrypted. But you can still see the TLS handshake, the SNI (Server Name Indication) showing which domain was requested, and the overall flow of the connection.

### 7 - Observe RDP Traffic

Since I was connected to VM1 via RDP, that traffic was already flowing. I filtered for it:

```
tcp.port == 3389
```

RDP generates a constant stream of traffic because it's sending the desktop display in real time. Even when I wasn't doing anything, there was a steady flow of packets - this is normal for RDP since it needs to keep the session alive.

### 8 - Network Security Group Rules

To test how Azure's firewall works, I went to the Azure portal and added an inbound rule on VM2's NSG to block ICMP traffic. Then I tried pinging VM2 again - the requests timed out, and in Wireshark I could see the Echo Requests going out but no Echo Replies coming back.

After removing the block rule, pings started working again immediately.

## Key Takeaways

- Wireshark is incredibly powerful for understanding what's actually happening on a network
- Encrypted protocols (SSH, HTTPS) protect the payload but you can still see connection metadata
- DNS queries are unencrypted by default, which is something to be aware of from a security perspective
- Azure NSGs work at the network level and immediately affect traffic flow when rules change
- Understanding these protocols is essential for troubleshooting network issues and security analysis
