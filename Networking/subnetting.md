### 1. What a "Default Route" Does (and Doesn't) Do

A default route (`0.0.0.0/0 via Next-Hop IP`) is strictly a **Layer 3 navigation decision**. It tells the router's software:
> *"If you receive a packet for a destination you don't recognize, hand it over to Router B."*

However, deciding *where* to send a packet is only half the battle. The router must still physically **transmit the bits across the wire**.
Because standard network cables use Ethernet (Layer 2), the router cannot just shoot a raw IP packet down the copper:

* It must wrap the IP packet into an **Ethernet frame**.
* An Ethernet frame **requires a Destination MAC address**.
* To get Router B's MAC address, Router A must send an **ARP (Address Resolution Protocol) request**.

---

### 2. The Failure: Why ARP Breaks if Subnets Don't Match

Suppose you connect Router A and Router B with a cable:

* **Router A's interface:** `10.0.0.1/24` (with a default route pointing to Router B: `192.168.2.1`)
* **Router B's interface:** `192.168.2.1/24`

When a packet needs to be forwarded:

1. **Router A checks its interface subnet:** Router A looks at its port and sees it belongs to `10.0.0.0/24`. This means Router A believes only IPs between `10.0.0.1` and `10.0.0.254` exist on this physical wire.
2. **The ARP refusal:** Router A tries to resolve the Next-Hop IP (`192.168.2.1`). Because `192.168.2.1` does not fall inside `10.0.0.0/24`, standard network operating systems refuse to generate an ARP request for it out of that port.
3. **The "Martian" Packet:** Even if you force the router to broadcast an ARP request onto the wire, Router B receives an ARP request coming from `10.0.0.1`. Router B sees this source IP is completely alien to its own configured subnet (`192.168.2.0/24`), flags it as a configuration error or spoofed packet (a "martian"), and **silently drops it**.

Because Router A never receives an ARP reply, it never learns Router B's MAC address. The packet is dropped in the queue.

---
