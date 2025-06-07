## Protocol

A **protocol** is a set of rules or guidelines that define how data is transmitted, received, and processed in a network or communication system. Protocols ensure that different devices and systems can communicate effectively, even if they are made by different manufacturers or use different technologies.


## Resources
- [YT Playlist - Networking Fundamentals](https://www.youtube.com/playlist?list=PLIFyRwBY_4bRLmKfP1KnZA6rZbRHtxmXi)
   - [Hub, Bridge, Switch, Router](https://youtu.be/H7-NR3Q3BeI?si=tR2Cehz4ls3uCt4m)
   - [OSI Model - Layer 1, 2 (Network Access Layers)](https://youtu.be/H7-NR3Q3BeI?si=tR2Cehz4ls3uCt4m)
   - [OSI Model - Layer 3(Internet), 4(Transport)](https://youtu.be/0aGqGKrRE0g?si=V2yKC54iKpmHM-3i)
   - [OSI Model - Layer 5, 6, 7 (Application Layer)](https://youtu.be/2iFFRqzX3yE?si=JJ_tmTmY6d-GMjYS)
   - [**How Data moves through the Internet**](https://youtu.be/YJGGYKAV4pA?si=DLCunOcloJ4hAnWi)


## OSI Model
![](https://pbs.twimg.com/media/Fhib8hlUoAEPq1U?format=jpg&name=4096x4096) 

- **Routing Table:** A database stored in routers and hosts that determines where packets should be forwarded based on the destination IP(**Internet Protocol**).
- **ARP(Address Resolution Protocol) Table:** Maps IP addresses to MAC addresses for devices on the *same local network*.
```bash
rudraksh @🔥:~$ arp
Address                  HWtype  HWaddress           Flags Mask     Iface
reliance.reliance        ether   b4:a7:c6:fc:c3:69   C              wlp1s0
```
- **MAC(Media Access Control) Address Table (Switch Table):** Used by switches to map MAC addresses to switch ports.
- **NAT(Network Address Translation) Table:** A NAT (Network Address Translation) table is a critical component that enables multiple devices on a private network to share a single public IP address when communicating with the internet. This translation mechanism is fundamental to modern networking, helping conserve IPv4 addresses while providing basic security through obscurity.

## Broadcast vs Multicast
| Feature          | Broadcast (IPv4 exclusive)                         | Multicast                          |  
|------------------|------------------------------------|------------------------------------|  
| **Target**       | All devices in a network           | Only subscribed devices            |  
| **Efficiency**   | Low (floods the network)           | High (only sends to interested receivers) |  
| **Address Range**| `255.255.255.255` (IPv4)           | `224.0.0.0 – 239.255.255.255` (IPv4) |  
| **Use Cases**    | ARP, DHCP, NetBIOS                 | Video streaming(only users that wants to watch the video), VoIP, IoT updates |  



## [Subnet Mask - Explained](https://youtu.be/s_Ntt6eTn94?si=72uI-Mc9dEy9JF2s&t=661) 

   - For IP address `208.36.47.102/29` :
   - `208.36.47.102` → `11010000.00100100.00101111.01100110`
   - **Subnet Mask** (`29-bit`) → `255.255.255.248` → `11111111.11111111.11111111.11111000`
   - First and Last address in a subnet are reserved for network and broadcast

   | Category      |    Answer     | Full IP (Binary) |
   |---------------|---------------|------------------|
   | Network       | 208.36.47.96  | 11010000.00100100.00101111.01100000   |
   | First Host    | 208.36.47.97  | 11010000.00100100.00101111.01100001   | 
   | Last Host     | 208.36.47.102 | 11010000.00100100.00101111.01100110   |
   | Broadcast     | 208.36.47.103 | 11010000.00100100.00101111.01100111   |
   | Next Subnet   | 208.36.47.104 | 11010000.00100100.00101111.01101000   |



```bash
# Network Interface: Docker virtual bridge (for container networking)
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        # IPv4 Configuration:
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255  
        # MAC Address:
        ether 02:42:0d:7b:c7:ae  
        # Transmission queue length (0 for virtual interfaces):
        txqueuelen 0  (Ethernet)  
        # Receive (RX) statistics:
        RX packets 0  bytes 0 (0.0 B)          # No packets received
        RX errors 0  dropped 0  overruns 0  frame 0  # Zero errors
        # Transmit (TX) statistics:
        TX packets 0  bytes 0 (0.0 B)          # No packets sent
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0  # Zero errors

# Network Interface: Wired Ethernet (inactive in this case)
enp2s0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 10:7d:1a:25:81:51  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)          # Interface is idle
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# Network Interface: Loopback (localhost)
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        # IPv4 and IPv6 loopback addresses:
        inet 127.0.0.1  netmask 255.0.0.0       # Standard IPv4 loopback
        inet6 ::1  prefixlen 128  scopeid 0x10<host>  # IPv6 loopback
        loop  txqueuelen 1000  (Local Loopback)
        # Loopback traffic (high counts are normal):
        RX packets 231780  bytes 22052130 (22.0 MB)  
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 231780  bytes 22052130 (22.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# Network Interface: Virtual bridge for KVM/libvirt (virtual machines)
virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:60:79:07  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)          # No VM traffic
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# Network Interface: Wireless (active Wi-Fi connection)
wlp1s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        # IPv4 Configuration:
        inet 192.168.29.36  netmask 255.255.255.0  broadcast 192.168.29.255  
        # IPv6 Addresses (multiple scopes):
        inet6 2405:201:303e:d849:2cde:72a0:8930:6435  prefixlen 64  scopeid 0x0<global>  # Public IPv6
        inet6 fe80::176f:2866:7939:2e26  prefixlen 64  scopeid 0x20<link>  # Link-local IPv6
        inet6 2405:201:303e:d849:6ef7:11ad:99fc:70e6  prefixlen 64  scopeid 0x0<global>  # Temporary IPv6
        # MAC Address:
        ether 54:13:79:5b:7f:7d  txqueuelen 1000  (Ethernet)
        # Active Traffic Statistics:
        RX packets 5014645  bytes 5415231927 (5.4 GB)  # Data received
        RX errors 0  dropped 0  overruns 0  frame 0    # No receive errors
        TX packets 1809024  bytes 521548988 (521.5 MB) # Data transmitted
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0  # No transmit errors
```

# IPv4 vs [IPv6](https://github.com/onemarcfifty/cheat-sheets/blob/main/networking/ipv6.md)

A packet is either **IPv4** or **IPv6**, never both.<br/>
**No NAT needed** due to abundant addresses. 


- IPv6 represented as **8 groups of 16 bits (hextets)**, written in **hexadecimal** and separated by colons (`:`).  
  ```
  2001:0db8:0000:0000:0000:ff00:0042:8329/64  
  ```  

Prefix (Network): The leftmost bits (typically /64 for most subnets).

Interface ID (Host): The rightmost 64 bits (often derived from MAC or random).

#### **Shortening Rules**  
1. **Remove leading zeros** in each hextet:  
   - `0042` → `42`  
   - `0000` → `0`  
   ```
   2001:db8:0:0:0:ff00:42:8329  
   ```  

2. **Replace consecutive zero groups with `::`** (once per address):  
   ```
   2001:db8::ff00:42:8329  
   ```  

#### **Special Addresses**  
- **Loopback** (like IPv4’s `127.0.0.1`):  
  ```
  ::1  
  ```  
- **Unspecified address** (like `0.0.0.0`):  
  ```
  ::  
  ```  

#### **IPv6 in URLs**  
- Enclose IPv6 addresses in **square brackets** to avoid confusion with port numbers:  
  ```
  http://[2001:db8::200e]  
  http://[2001:db8::200e]:8080/path  
  ```  
### **Transition Mechanisms** 
**IPv6 packets** are **dropped** by **IPv4**-only devices **unless translation** (NAT64) or tunneling (6to4) is used.
Many older devices still lack IPv6, especially in legacy enterprise/IoT systems.eg. Windows XP (without SP3, a major update to the Windows XP operating system) .
   - Techniques like **6to4**, **Teredo**, or **NAT64** translate between IPv4 and IPv6, but the packet itself remains either IPv4 **or** IPv6.  
   - Example: NAT64 converts an IPv6 packet to IPv4 at a gateway, but the original packet doesn’t contain both headers.  

| **Method**               | **Format**                     | **Purpose**                          |
|--------------------------|-------------------------------|--------------------------------------|
| IPv4-Mapped IPv6         | `::FFFF:<IPv4>`              | IPv6 apps to IPv4 hosts              |
| 6to4 Addressing         | `2002:<IPv4-hex>::/48`       | Tunneling over IPv4 networks         |
| NAT64/DNS64             | `64:FF9B::<IPv4>`            | IPv6-only clients to IPv4 servers    |

These mappings ensure backward compatibility and smooth transition from IPv4 to IPv6. The most commonly used today are **IPv4-mapped IPv6 addresses** and **NAT64/DNS64** for translation.

## **[SLAAC](https://en.wikipedia.org/wiki/IPv6#Stateless_address_autoconfiguration_(SLAAC)) (Stateless address autoconfiguration)**

**SLAAC** (Stateless Address Autoconfiguration) is a method used by IPv6 hosts to automatically configure their own IP addresses without the need for a DHCP (Dynamic Host Configuration Protocol) server. It is defined in **RFC 4862**.

### **How SLAAC Works:**
1. **Router Advertisement (RA) Messages**  
   - An IPv6 router periodically sends **Router Advertisement (RA)** messages to all hosts on the local network (via multicast).
   - These messages contain:
     - The network prefix (e.g., `2001:db8::/64`).
     - Default gateway information.
     - Flags indicating whether DHCPv6 is needed for additional configuration (like DNS).

2. **Generating an Interface Identifier**  
   - The host takes the network prefix from the RA and generates a unique **Interface Identifier (IID)** for its own address.
   - The IID can be created in two ways:
     - **EUI-64 Method**: Uses the device's MAC address (flips a bit and inserts `FFFE`).
     - **Privacy Extensions (RFC 4941)**: Randomly generated to enhance privacy.

3. **Forming the IPv6 Address**  
   - The host combines the network prefix and its IID to form a complete IPv6 address (e.g., `2001:db8::1a2b:3c4d:5e6f:7a8b`).

4. **Duplicate Address Detection (DAD)**  
   - Before using the address, the host checks for duplicates by sending a **Neighbor Solicitation (NS)** message to ensure no other device has the same address.

### **Key Features of SLAAC:**
- **Stateless**: No server maintains address assignments (unlike DHCPv6).
- **Automatic**: Hosts configure themselves without manual input.
- **Supports Multiple Addresses**: A host can have both SLAAC and DHCPv6 addresses.
- **No DNS Configuration by Default**: SLAAC only assigns IP addresses; DNS settings usually require DHCPv6 or manual configuration (unless using **RA options like RDNSS**).


### **Example of SLAAC Address:**
- **Network Prefix (from RA):** `2001:db8:1234::/64`  
- **Interface ID (EUI-64 from MAC `00:1A:2B:3C:4D:5E`):** `021a:2bff:fe3c:4d5e`  
- **Final IPv6 Address:** `2001:db8:1234:0:21a:2bff:fe3c:4d5e`  

### **Limitations:**
- Does not provide DNS server information by default (unless using **RFC 6106** for DNS options in RA).
- Less control over address assignment compared to DHCPv6.

---

### **DHCPv6 (Dynamic Host Configuration Protocol for IPv6)**  
**DHCPv6** is a network protocol used to automatically assign IPv6 addresses and other network configuration parameters (like DNS servers) to hosts in an IPv6 network. Unlike **SLAAC** (which is stateless), DHCPv6 can operate in **stateful** or **stateless** mode, providing more control over address assignment and additional configuration options.

| Mode | Description | Use Case |
|------|------------|----------|
| **Stateful DHCPv6** | The server assigns IPv6 addresses and other info (DNS, domain names). Similar to DHCP in IPv4. | When precise control over IP assignments is needed (e.g., enterprise networks). |
| **Stateless DHCPv6** | Hosts use **SLAAC** for addresses but get DNS, NTP, etc., from DHCPv6. | When automatic IP assignment (via SLAAC) is sufficient, but extra settings (like DNS) are needed. |


### **DHCPv6 Message Exchange (Stateful)**
When a host boots up, it performs the following steps:

1. **Router Solicitation (RS)** *(Optional)*  
   - The host sends an **RS** to discover routers.
2. **Router Advertisement (RA)**  
   - A router replies with an **RA**, indicating whether to use **SLAAC**, **DHCPv6**, or both.  
     - **RA Flag `M` (Managed)**: "Use DHCPv6 for addresses."  
     - **RA Flag `O` (Other)**: "Use DHCPv6 for other settings (DNS, etc.)."  
3. **DHCPv6 Solicit (Client → Server)**  
   - The host sends a **Solicit** message (multicast to `ff02::1:2`) to find DHCPv6 servers.  
4. **DHCPv6 Advertise (Server → Client)**  
   - Available servers respond with an **Advertise** message.  
5. **DHCPv6 Request (Client → Server)**  
   - The client requests an IP from a chosen server.  
6. **DHCPv6 Reply (Server → Client)**  
   - The server assigns the IP and sends configuration details (DNS, domain, etc.).  

*(Stateless DHCPv6 skips address assignment and only provides extra info.)*

---

### **Key Features of DHCPv6**
- **IPv6 Address Assignment** (Stateful mode only).  
- **DNS, NTP, SIP, and Domain Name Configuration** (Both stateful & stateless).  
- **Lease Management** (Like IPv4 DHCP, with renewal & rebinding).  
- **Prefix Delegation (DHCPv6-PD)** – Used by ISPs to assign entire subnets to customers (e.g., for home routers).  

### **Common DHCPv6 Ports & Multicast Addresses**
- **UDP Port 546** (DHCPv6 Client)  
- **UDP Port 547** (DHCPv6 Server)  
- **Multicast Address `ff02::1:2`** (All DHCPv6 servers/relay agents)  

---


# [DHCP](https://youtu.be/Cy0M54GSpBg?si=-b7AkNARhvwtkE19) (Dynamic Host Configuration Protocol)

![](https://info.teledynamics.com/hs-fs/hubfs/blog-images/DHCP%20server%20to%20client%20image.png?width=740&height=635&name=DHCP%20server%20to%20client%20image.png)

When you connect to a router (via Ethernet or Wi-Fi), the process involves **DHCP** (to get an IP) and **ARP** (to resolve MAC addresses). 

---

### **1. DHCP Discovery (First Step)**
When a device (client) connects to a router, it first needs an **IP address** via DHCP (if not manually configured). The DHCP process involves **broadcast** messages:

1. **DHCP Discover** (Broadcast)  
   - The client sends a **DHCP Discover** message to find a DHCP server (router).  
   - Since the client doesn't know the DHCP server's MAC or IP yet, it sends this as a **broadcast**:
     - **Source MAC**: Client's MAC  
     - **Destination MAC**: `FF:FF:FF:FF:FF:FF` (broadcast)  
     - **Source IP**: `0.0.0.0` (client has no IP yet)  
     - **Destination IP**: `255.255.255.255` (broadcast IP)  

2. **DHCP Offer (Router → Client)**  
   - The router (DHCP server) responds with a **DHCP Offer**, also sent as a **broadcast** (or unicast in some cases).  
   - The router knows the client's MAC from the **DHCP Discover** frame.  

3. **DHCP Request (Client → Router)**  
   - The client broadcasts a **DHCP Request** (accepting the offer).  
   - Still uses `FF:FF:FF:FF:FF:FF` as the destination MAC.  

4. **DHCP Ack (Router → Client)**  
   - The router confirms with a **DHCP Ack**, assigning the IP.  

At this point, the client has an **IP address**, **subnet mask**, **default gateway (router's IP)**, and **DNS servers**.

---

### **2. ARP Request (After DHCP, exclusive for IPv4)**
Now that the client has the **router's IP** (default gateway), it may need the **router's MAC address** for further communication (e.g., pinging the internet).  

1. **Client checks its ARP cache** (does it already know the router's MAC?).  
   - If not, it sends an **ARP Request**:  
     - **Source MAC**: Client's MAC  
     - **Destination MAC**: `FF:FF:FF:FF:FF:FF` (broadcast)  
     - Message: *"Who has <router's IP>? Tell <client's MAC>"*  

2. **Router Responds with ARP Reply**  
   - The router replies with its MAC address.  
   - Now, the client knows the router's MAC and can send traffic to it.  

If you check an **IPv6** network, you won’t find an "ARP table." Instead, use commands like:

`show ipv6 neighbors` (Cisco) or `ip -f inet6 neigh` (Linux) to see the NDP cache.NDP also includes features like Duplicate Address Detection (DAD) and Router Discovery, which ARP lacks.

---

### **What *Could* Be Done (But Isn’t Standard)**

- DHCP **could** define a new **custom option** (e.g., "Gateway MAC Address") and include it in the Offer/Ack.  

**Why Doesn’t DHCP Just Include the *Gateway’s* MAC?**

Even if DHCP included the **gateway’s MAC** (instead of the DHCP server’s MAC bc the DHCP Server Might Not Be the Router), problems remain:  
1. **The client still must ARP for other devices** (not just the gateway).  
2. **The gateway’s MAC might change** (e.g., router replacement, failover in HA setups).  
3. **Unnecessary complexity** — ARP already solves this elegantly. Keeping them separate follows the **separation of concerns** principle in networking.
4. **Security & Spoofing Risks** - If DHCP provided MAC addresses, attackers could spoof DHCP packets to poison MAC tables (similar to ARP spoofing).
ARP has mechanisms (like gratuitous ARP) to detect conflicts, but DHCP doesn’t.

---

# [DNS](https://youtu.be/27r4Bzuj5NQ?si=7rKZOof-6e1YfFuR&t=17)(Domain Name Server)

1. **Recursive Resolver (DNS Resolver)**  
   - Queries authoritative servers on behalf of the client.
   - Caches responses to reduce lookup time.
   - Example: ISP’s DNS server, Google DNS (`8.8.8.8`), Cloudflare (`1.1.1.1`).

2. **Root Name Server**  
   - Directs queries to TLD servers.
   - There are **13 root server clusters** worldwide.

3. **TLD(Top Level Domain) Name Server**  
   - Manages domain extensions (e.g., `.com`, `.org`).
   - Directs queries to authoritative name servers.

4. **Authoritative Name Server**  
   - Contains the actual DNS records for a domain.
   - Responds with the final IP address.(www.example.com → 192.0.2.1)

![](https://www.indusface.com/wp-content/uploads/2024/10/DNS-lookup-process-.png)

## **DNS-over-HTTPS (DoH)**
**DOH** is a security protocol that encrypts DNS queries using **HTTPS** (the same protocol used for secure web browsing). It prevents eavesdropping and manipulation of DNS traffic by third parties (e.g., ISPs, hackers, governments).

- Resistant to **DNS spoofing, censorship, and surveillance**.
- Supported by major browsers (Firefox, Chrome, Edge) and DNS providers (Cloudflare, Google).

### **DoH Lookup Process:**
1. **Client (Browser/OS)** sends a DNS query (e.g., `example.com`).
2. Instead of using standard **DNS (UDP 53)**, the request is sent via **HTTPS (port 443)** to a **DoH-compatible resolver** (e.g., Cloudflare `1.1.1.1`).
3. The DoH resolver processes the query and returns the encrypted response.
4. The client decrypts the response and connects to the resolved IP.


## **How to Enable DoH?**
### **A. In Web Browsers**
#### **Firefox**
1. Go to `about:config`.
2. Search for `network.trr.mode`.
3. Set to `2` (enable DoH by default).
4. Set `network.trr.uri` to a DoH provider (e.g., `https://cloudflare-dns.com/dns-query`).

#### **Chrome/Edge**
1. Go to `Settings > Privacy and Security > Security`.
2. Enable **"Use secure DNS"** and select a provider (e.g., Cloudflare, Google).

### **B. At OS Level (Windows 11)**
1. Go to **Settings > Network & Internet > Ethernet/Wi-Fi**.
2. Click **"Edit DNS Settings"** → **"Encrypted Only (DNS-over-HTTPS)"**.
3. Enter a DoH server (e.g., `https://dns.google/dns-query`).

### **C. Using Public DoH Resolvers(free of cost)**
| Provider | DoH Server URL |
|----------|----------------|
| **Cloudflare** | `https://cloudflare-dns.com/dns-query` |
| **Google** | `https://dns.google/dns-query` |
| **Quad9** | `https://dns.quad9.net/dns-query` |
| **NextDNS** | `https://dns.nextdns.io/your-id` |
---

## **DNS-over-TLS (DoT)**
| Factor | DoT (DNS-over-TLS) | DoH (DNS-over-HTTPS) |
|--------|---------------------|-----------------------|
| **Port** | Dedicated (853) | Shared (443) |
| **Traffic Pattern** | Clearly DNS | Looks like normal HTTPS |
| **Blocking Method** | Simple port block | Needs DPI(Deep Packet Inspection)/ML |

---

# socket programming L20

# [ICMP](https://youtu.be/5EP1NZAu_PQ?si=JXkPF8yO30COuqdZ)

[General structure of ICMPv6 Messages](https://en.wikipedia.org/wiki/ICMPv6#Message_types_and_formats)

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e1/ICMP_header_-_General-en.svg/330px-ICMP_header_-_General-en.svg.png)


An **ICMP (Internet Control Message Protocol)** *packet* is generated in various scenarios to report network conditions or errors. Here are common cases:

1. **Ping Requests/Replies** – When using the `ping` command, ICMP Echo Request (Type 8) and Echo Reply (Type 0) packets are exchanged.
2. **Destination Unreachable** – If a router or host cannot deliver a packet, it sends an ICMP **Destination Unreachable (Type 3)** message.
3. **Time Exceeded** – When a packet’s TTL (Time to Live, gets decremented by 1 at each hop) reaches zero, a **Time Exceeded (Type 11)** message is sent (used in `traceroute`).
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
# Popular routing protocols - L24 

# firewall, ddos....

# future - sdns - last lecture




## [Transmission Control Protocol(TCP)](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)  VS [User Datagram Protocol(UDP)](https://en.wikipedia.org/wiki/User_Datagram_Protocol)

Port numbers depends on the top most Service/Application.

![](https://www.metered.ca/blog/content/images/2024/10/tcp-vs-udp-1.png)
