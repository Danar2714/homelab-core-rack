# Router Mikrotik hEX RB750Gr3
# A. Base Configuration

## 1. Initial connection using MAC address

To perform the very first configuration, the router was accessed from a Windows laptop using **WinBox** and the device’s **MAC address** instead of its IP.  
This avoids any dependency on the default IP configuration and works even if the IP addressing has not yet been set.

<img src="../docs/init_connection.png" alt="Initial WinBox connection to the hEX router using MAC address" width="50%" />

**Figure 1 – Initial connection via MAC address**  
WinBox is used to connect to the router by selecting the device from the “Neighbors” tab and authenticating with the default admin credentials.

---

## 2. Reviewing the factory default configuration

After the first login, RouterOS displays the **Default Configuration** window.  
Here you can see the vendor’s preset configuration, which typically includes a basic WAN–LAN setup, DHCP server on the LAN bridge, firewall rules, and NAT.

<img src="../docs/default_config.png" alt="RouterOS default configuration summary window" width="50%" />

**Figure 2 – Factory default configuration**  
The summary shows that ether1 is used as the WAN port and the remaining Ethernet interfaces are part of a pre-configured LAN bridge with DHCP and firewall enabled.

---

## 3. Creating the LAN bridge

A dedicated bridge called **`bridge-lan`** was created to aggregate the internal LAN interfaces.  
This bridge will later include all LAN ports (ether2–ether5), keeping **ether1** reserved for the WAN/Internet uplink.

<img src="../docs/bridge-lan_creation3.png" alt="Creating a new bridge interface from the Interfaces menu" width="50%" />

**Figure 3 – Creating a bridge interface**  
From the *Interfaces* menu, the **Bridge** type is selected to create a new logical interface that will act as the LAN bridge.

<img src="../docs/bridge-lan_creation.png" alt="bridge-lan interface created in the Bridge menu" width="50%" />

**Figure 4 – bridge-lan created**  
The new `bridge-lan` interface appears in the *Bridge* menu with RSTP enabled, ready to receive member ports.

---

## 4. Adding LAN ports to the bridge

All internal LAN ports (**ether2**, **ether3**, **ether4**, **ether5**) are added as **bridge ports** to `bridge-lan`.  
The WAN port (**ether1**) is intentionally left **outside** the bridge so it can be used as the Internet uplink.

<img src="../docs/add_to_bridge.png" alt="Adding ether2 as a port member of bridge-lan" width="50%" />

**Figure 5 – Adding ports to the bridge**  
Each Ethernet interface is attached to `bridge-lan` using the *Bridge Port* configuration, keeping hardware offload enabled for better performance.

---

## 5. Verifying bridge results

After all LAN interfaces have been added, the *Ports* tab of the Bridge menu shows `bridge-lan` with **ether2–ether5** as member ports.  
This confirms that all internal switch ports now belong to the same LAN segment, while ether1 remains separate for WAN/Internet.

<img src="../docs/bridge_result.png" alt="Bridge ports table showing ether2–ether5 as part of bridge-lan" width="50%" />

**Figure 6 – Final bridge configuration**  
The bridge port table lists each LAN port (ether2–ether5) as a designated port on `bridge-lan`, completing the basic LAN aggregation.
