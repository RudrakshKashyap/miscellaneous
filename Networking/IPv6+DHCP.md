# IPv4 vs [IPv6](https://www.youtube.com/watch?v=z7Al3P8ShM8)

A packet is either **IPv4** or **[IPv6](https://github.com/onemarcfifty/cheat-sheets/blob/main/networking/ipv6.md)**, never both.<br/>
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
