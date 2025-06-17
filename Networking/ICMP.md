# [ICMP](https://youtu.be/5EP1NZAu_PQ?si=JXkPF8yO30COuqdZ)

[General structure of ICMPv6 Messages](https://en.wikipedia.org/wiki/ICMPv6#Message_types_and_formats)

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e1/ICMP_header_-_General-en.svg/330px-ICMP_header_-_General-en.svg.png)


An **ICMP (Internet Control Message Protocol)** *packet* is generated in various scenarios to report network conditions or errors. Here are common cases:

1. **Ping Requests/Replies** – When using the `ping` command, ICMP Echo Request (Type 8) and Echo Reply (Type 0) packets are exchanged.
   ```bash
   rudraksh @🔥:~$ping google.com -c 2
   PING google.com (2404:6800:4002:802::200e) 56 data bytes
   64 bytes from del03s01-in-x0e.1e100.net (2404:6800:4002:802::200e): icmp_seq=1 ttl=118 time=49.1 ms
   64 bytes from del03s01-in-x0e.1e100.net (2404:6800:4002:802::200e): icmp_seq=2 ttl=118 time=48.2 ms

   --- google.com ping statistics ---
   2 packets transmitted, 2 received, 0% packet loss, time 1002ms
   rtt min/avg/max/mdev = 48.168/48.648/49.129/0.480 ms
   ```
2. **Destination Unreachable** – If a router or host cannot deliver a packet, it sends an ICMP **Destination Unreachable (Type 3)** message.
3. **Time Exceeded** – When a packet’s TTL (Time to Live, gets decremented by 1 at each hop) reaches zero, a **Time Exceeded (Type 11)** message is sent (used in `traceroute`).

   ```bash
   rudraksh @🔥:~$mtr google.com #mytraceroute
   rudraksh-Inspiron-15-3567 (2405:201:303e:d849:3814:bc662025-06-08T18:22:19+0530
   Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                          Packets               Pings
   Host                                Loss%   Snt   Last   Avg  Best  Wrst StDev
   1. (first hop, local router, 2405:201:303e:d849:b6a7:c6ff:fef)
   1. 2405:201:303e:d849:b6a7:c6ff:fef  0.0%     7    7.1   3.4   1.9   7.1   2.2
   2. (waiting for reply)
   3. 2405:200:801:1200::1004           0.0%     7   23.2  22.5  21.7  23.2   0.6
   4. 2405:200:801:1200::105a           0.0%     7   21.6  20.3  19.1  21.6   0.9
   5. 2405:200:870:3168:61::4           0.0%     7   18.0  19.2  18.0  20.2   0.8
   6. (waiting for reply)
   7. 2405:200:801:1200::11da           0.0%     7   23.0  22.9  21.2  24.3   1.1
   8. (waiting for reply)
   9. 2404:6800:8281:c0::1              0.0%     7   39.8  44.8  39.8  70.1  11.2
   10. 2404:6800:8281:c0::1              0.0%     7   38.5  39.9  38.3  43.0   1.7
   11. 2404:6800:8281:c0::1              0.0%     7   39.1  41.1  38.5  46.3   2.8
   12. 2001:4860:0:1::7ba8              80.0%     6  109.0 109.0 109.0 109.0   0.0
   13. 2001:4860:0:1::1bae               0.0%     6   39.9  50.3  39.9  96.7  22.7
   14. 2001:4860::9:4001:ddce            0.0%     6   50.7  61.6  50.7  78.7  11.9
   15. 2001:4860::9:4001:67bd            0.0%     6   53.1  65.3  51.3 111.7  24.0
   16. 2001:4860:0:1::77ad              33.3%     6   54.3  54.7  52.5  58.8   2.8
   17. 2001:4860:0:1::5505               0.0%     6   46.3  48.3  46.3  51.9   2.5
   18. del03s05-in-x0e.1e100.net         0.0%     6   46.9  48.9  46.4  52.1   2.5
   18. (Google-owned server (final hop in your traceroute), del03s05-in-x0e.1e100.net)
   ```

4. **Redirect Message** – A router may send an ICMP **Redirect (Type 5)** to suggest a better route.
5. **Fragmentation Needed** – If a packet requires fragmentation but the DF (Don’t Fragment) flag is set, an ICMP **Fragmentation Needed (Type 3, Code 4)** message is sent.

ICMP helps diagnose and manage IP communication but does not carry application data. Some ICMP messages may be blocked by firewalls for security.

### How will a router knows that the packet it sent got lost in void?

A router **typically doesn’t know** if a forwarded packet is lost(so no ICMP packet is generated) unless:
1. An ICMP error is returned (rare for random drops).
2. A routing protocol notifies it of a link failure. (Long-Term Detection)
3. External tools (like `ping`, `traceroute`) are used to probe the path.

Some other cases when an **ICMP packet is not generated** to prevent network congestion, security risks, or protocol misuse:  

- **1. For Another ICMP Error Message**  
   If an ICMP packet (e.g., "Destination Unreachable") itself fails, **no ICMP error is sent** to avoid infinite loops. Suppose the TTL of ICMP packet itsefl has reached 0 and the router decides to drop it, then it will not send another ICMP packet to source.

-  **2. For Non-Initial IP Fragments**  
   If a fragmented IP packet (not the first fragment) is dropped, **no ICMP error is sent**.  

-  **3. For Broadcast/Multicast Traffic**  
   ICMP errors are **not sent** in response to broadcast or multicast packets (to prevent flooding).  

-  **4. For Packets with Invalid Source IPs**  
   If the source IP is **zero (0.0.0.0)**, **loopback (127.x.x.x)**, or a **non-routable address**, ICMP errors are **not generated**.  

-  **5. For Certain Security/Drop Policies**  
   - Firewalls/routers may **silently drop** packets (without ICMP errors) due to:  
     - **Rate-limiting** (to prevent ICMP flooding).  
     - **Administrative filtering** (e.g., DDoS protection).  
     - **Privacy/Security** (e.g., "stealth" mode to avoid network scanning).  

-  **6. For Some Link-Layer Errors**  
   If a packet is dropped at **Layer 2 (e.g., Ethernet/Wi-Fi)**, no ICMP message is sent (ICMP operates at **Layer 3**).  



## **ICMP in IPv6 (ICMPv6)**
- **RFC**: 4443 (replaces ICMP for IPv6).
- **New Features**:
  - **Neighbor Discovery Protocol (NDP)** (replaces ARP).
  - **Multicast Listener Discovery (MLD)**.
  - **Router Solicitation/Advertisement**.
- **Key Differences**:
  - New message types (e.g., **Router Solicitation (133)**, **Neighbor Advertisement (136)**).
  - No more "Source Quench" (replaced by better congestion control mechanisms).

---