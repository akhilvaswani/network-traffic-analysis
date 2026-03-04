# Network Traffic Analysis

Set up two VMs in Azure (Windows 10 + Ubuntu Server) on the same virtual network and used Wireshark to capture and analyze different types of network traffic. Went through ICMP, SSH, DNS, HTTP/HTTPS, and RDP to see what each protocol actually looks like on the wire.

## Setup

- VM1 (Windows 10) -- ran Wireshark here to do the captures
- VM2 (Ubuntu Server) -- target machine for pings, SSH, etc.
- Both on the same Azure subnet so traffic stayed local

## What I looked at

**ICMP (ping)** -- Pinged VM2 from VM1 and watched the echo request/reply pairs in Wireshark. Also pinged google.com to compare -- local pings were under 1ms, external were around 10-15ms.

**SSH** -- SSH'd into VM2 and filtered on `tcp.port == 22`. You can see the TCP handshake and key exchange but the actual data shows up as "Encrypted packet" which is expected.

**DNS** -- Used `nslookup` to query a few domains and filtered on `dns`. Could see the full query/response -- the request going out asking for an IP and the A record coming back.

**HTTP/HTTPS** -- Browsed some websites and filtered on ports 80 and 443. Most traffic these days is HTTPS so the payload is encrypted, but you can still see the TLS handshake and the SNI field showing which domain was requested.

**RDP** -- Since I was connected via RDP, that traffic was already there. Filtered on `tcp.port == 3389`. RDP generates a constant stream of packets even when you're not doing anything because it's always updating the display.

## NSG testing

Also tested Azure's Network Security Group by adding a rule to block ICMP on VM2. Pings immediately started timing out -- in Wireshark I could see the requests going out but no replies coming back. Removed the rule and it started working again right away. Pretty cool to see the firewall take effect in real time.

## Takeaways

- Wireshark is a lot more useful once you know what filters to use
- Even encrypted protocols (SSH, HTTPS) expose connection metadata
- DNS queries are unencrypted by default which is worth knowing
- Azure NSG rule changes take effect immediately
