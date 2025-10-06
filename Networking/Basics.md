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
rudraksh @🔥:~$arp
Address                  HWtype  HWaddress           Flags Mask     Iface
reliance.reliance        ether   b4:a7:c6:fc:c3:69   C              wlp1s0
```
- **MAC(Media Access Control) Address Table (Switch Table):** Used by switches to map MAC addresses to switch ports.
- **[NAT(Network Address Translation) Table:](https://www.youtube.com/playlist?list=PLIFyRwBY_4bQ7tJvbLA9A0v8Fq9l-H923)** A NAT (Network Address Translation) table is a critical component that enables multiple devices on a private network to share a single public IP address when communicating with the internet. This translation mechanism is fundamental to modern networking, helping conserve IPv4 addresses while providing basic security through obscurity.

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


## Common Commands

### **`netstat`**  
- **Function:** Shows active network connections, listening ports, routing tables, and interface statistics.  
- **Use Case:** Identifying open ports, checking for unauthorized connections.  
- **Example:**  
  - `netstat -a` (all connections)  
  - `netstat -r` (routing table) .  

### **`ss`**  (Modern netstat alternative in Linux)
- **Function:** Displays socket statistics (open ports, connections).  
- **Use Case:** Checking active connections more efficiently than `netstat`.  
- **Example:** `ss -tuln` (listening TCP/UDP ports) .  

### **`curl`**  
- **Function:** Transfers data from or to a server using various protocols (HTTP, HTTPS, FTP, SFTP, SCP, SMTP, and more).  
- **Use Case:** Testing APIs, checking web server responses.  
- **Example:** 
   - `curl -I http://google.com` (Send a GET Request & Display Headers only)
   - `curl -X POST -d "name=John&age=30" https://example.com/api` (Send a POST Request with Data)

### **`nmap`**  
- **Function:** Scans networks for open ports and services.  
- **Use Case:** Security auditing and network discovery.  
- **Example:** `nmap -sP 192.168.1.0/24` (ping scan) .  



### **`ifconfig`**  
```bash
rudraksh @🔥:~$ifconfig
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





# Popular routing protocols - L24 

# firewall, ddos....

# future - sdns - last lecture
