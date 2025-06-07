Here’s a breakdown of the key terms related to Wi-Fi(**Wireless Fidelity**) networks:

### **Beacons**
   - **Definition**: Beacons are management frames broadcast by Access Points (APs) to announce their presence and provide essential network information. They are transmitted periodically (e.g., every 100 ms by default) and include details like the SSID, BSSID (AP's MAC address), supported security protocols (WPA2, WPA3), channel, and data rates .
   - **Purpose**: Enable passive scanning by devices to discover nearby networks without actively probing. They also synchronize connected devices and manage power-saving modes via the Traffic Indication Map (TIM) .
   - **Hidden SSIDs**: Some APs can hide the SSID in beacon frames (showing a null/empty SSID), but this is not a security measure and can cause connectivity issues .

### **Access Point (AP)**
   - A hardware device (e.g., router or dedicated AP) that creates a wireless network by transmitting and receiving Wi-Fi signals. 
   

### **SSID (Service Set Identifier)**
   - **Definition**: The human-readable name of a Wi-Fi network (e.g., "HomeWiFi"). It’s included in beacons and probe responses .
   - **Hidden SSIDs**: APs can omit the SSID in beacons, requiring clients to send directed probe requests with the exact SSID to connect. However, this is ineffective for security and can be detected via probe requests from clients .

### **Probe Request**
   - **Definition**: A management frame sent by client devices (e.g., smartphones) to actively discover nearby networks. It can be:
     - **Directed**: Contains a specific SSID the device is seeking (e.g., "Starbucks_WiFi").
     - **Broadcast/Wildcard**: Requests any available network (SSID field is empty) .
   - **Purpose**: Faster than passive scanning, as it triggers APs to respond immediately with a probe response (similar to a beacon but unicast) .
   - **Security Risks**:
     - Devices leak saved SSIDs in probe requests, revealing past network connections and potential location history .
     - Attackers can spoof SSIDs from probe requests to create rogue APs .

### Additional Notes:
- **Active vs. Passive Scanning**: Devices use both methods—passive (listening for beacons) and active (sending probe requests)—to discover networks .
- **MAC Randomization**: Modern devices randomize MAC addresses in probe requests to prevent tracking, though SSID leaks remain a concern.

---

### WLAN (Wireless Local Area Network)
A LAN that uses Wi-Fi instead of wired Ethernet for device connections.

### VLAN (Virtual Local Area Network)
A logically segmented network within a physical LAN/WLAN, created using software (not hardware).  <br />Improves security, reduces broadcast traffic, and isolates groups (e.g., separating guest traffic from employees).

# TODO - authentication -> WPA3
