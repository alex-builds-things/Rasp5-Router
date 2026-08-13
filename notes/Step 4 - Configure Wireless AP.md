# Configuring Wireless Access Point (AP)
These instructions will be written generically, the specific menu labels and navigation paths will vary between AP brands and firmware versions. The settings and outcomes required are the same regardless of which AP you use.

# 1.1 The APs job in this setup
The AP takes wi-fi traffic from two separate wireless networks, tags each packet with the correct VLAN ID, and sends all of it upstream tot he Pi over a single ethernet cable. The Pi handles all routing, NAT, DHCP, and firewall decision. The AP does not route, NAT, or assign IP addresses, it is a wireless broadcaster only.

|   What AP handles |   What the Pi Handles |
|--------------------|----------------------|
| Wireless client connections | Routing between VLANs and internet  |
| VLAN tagging per SSID | NAT masquerade to ISP |
| 802.1Q trunk on uplink port   | DHCP address assignment   |
| Nothing else  | Firewall and VLAN isolation   |


# 1.2 Note the following

    - The AP must be powered on and connected to your existing home network for initial configuration.
    - Find the AP's IP address on your home network.
    - Have a browser ready to access the AP's web management interface
    - Know the AP's default admin credentials, or set custom admin credentials.

    - Some AP interfaces have two separate VLAN sections: one for Management VLAN (which controls which VLAN reaches the AP admin interface itself) and one for SSID VLAN assignment, which tags wireless traffic. For this project, only configure the SSID VLAN section. Changing the Management VLAN without a properly configured trunk connection will lock you out the web interface immediately.


# 1.3 Initial Access and Setup
    
        1. Open a browser and navigate to the AP's IP address
        2. Log in with the admin credentials.
        3. If the AP offers a setup wizard, work throught it. Key decisions during the wizard include:
           A. Select Standalone Mode, Access Point Mode, or equivalent, not controller mode or router mode.
           B. If asked about DHCP. disable it, as the Pi handles DHCP
           C. If asked about upstream DNS, leave it as default, as the Pi handles DNS
        4. Confirm the dashboard loads fully before making any changes.


# 1.4 Set the AP to Access Point or Standalone Mode
Some dedicated APs, such as ceiling-mount enterprise-style units, are always in AP mode and have no router mode to disable. If your AP has no such toggle, it is already operating correctly and you can skip this section.

If not, you must ensure that the AP operates as a pure wireless bridge, not as a router. Look for this setting under one of these labels depending on your AP brand:
        A. Access Point Mode
        B. Standalone Mode
        C. Bridge Mode
        D. AP Only Mode

When this mode is active, the AP disables it own NAT, routing, and DHCP server. If your AP was originally a consumer router (such as an older NetGear or Linksys device), enabling AP mode diables its routing functions.


# 1.5 Create SSID for VLAN 10 Devices
Create the first wireless network that VLAN 10 devices will connect to. Look for Wireless, Wireless Networks, SSIDs, or Wi-Fi networks in the menu.

Create a new wireless network with the following settings/information:

| Setting   | Value to configure    |
|----------|-------|
| SSID / Network Name   |   Any name you prefer for VLAN 10 devices |
| VLAN ID   |   10  (This is critical to ensure that all traffic from this SSID is tagged correctly)    |
|   Security    |   WPA2-Personal or WPA3-Personal  |
|   Password    |   Set a strong password, and record it    |
|Frequency Band |   2.4 GHz and 5 GHz for VLAN 10 devices   |
| DHCP  |   Disable it  |

Save the settings and confirm that the dashboard is still accessible.

# 1.6 Create SSID for VLAN 20 Devices
Create the first wireless network that VLAN 20 devices will connect to. Look for Wireless, Wireless Networks, SSIDs, or Wi-Fi networks in the menu.

Create a new wireless network with the following settings/information:

| Setting   | Value to configure    |
|----------------|-------------------|
| SSID / Network Name   |   Any name you prefer for VLAN 20 devices |
| VLAN ID   |   20  (This is critical to ensure that all traffic from this SSID is tagged correctly)    |
|   Security    |   WPA2-Personal or WPA3-Personal  |
|   Password    |   Set a strong password, and record it    |
|Frequency Band |   2.4 GHz for VLAN 20 devices   |
| DHCP  |   Disable it  |

Save the settings and confirm that the dashboard is still accessible.


# 1.7 Client Isolation on VLAN 20 (If available)
Client isolation, also known as AP isolation or wireless isolation, prevents devices on the same SSID from communicating directly with each other over wi-fi. When enabled on the VLAN 20 SSID, these devices will only be capable of communicating with the Pi gateway.

Look for this setting under VLAN 20's SSID network advanced or security settings. It maybe labeled:
    - Client Isolation
    - AP Isolation
    - Wireless Isolation
    - Station Isolation

If found, enable it on the VLAN 20 SSID only. The nftables rules on the Pi enforce VLAN 20 to VLAN 10 isolation at the network level regardless of whether client isolation is available on the AP.

# 1.8 Uplink Port VLAN Configuration (If available)
Some APs require the uplink ethernet port to be explicitly configured as a trunk before they will send tagged traffic to the Pi. Others handle this automatically when SSIDs are VLAN-tagged. Look in the AP interface for any of these sections:
    - Network -> LAN -> PORT VLAN
    - WAN/Uplink Settings
    - Port Settings -> VLAN
    - Advanced -> VLAN -> Tagged Ports

If uplink or VLAN configuration section is present, configure it as follows:

|   Settings    |   Value to Configure  |
|-----------------|----------------------|
| Port Mode |   Trunk   |
| Tagged VLANs  |   VLAN 10 and VLAN 20 (both must be added as tagged)  |
| Untagged/Native VLAN  |   Remove or leave unset   |
| Management VLAN   |   Do not change   |

If no such section exists, the AP handles trunk configuration automatically based on the SSID VLAN assignments.


# 1.9 Connect the AP to the Pi

1. Unplug the ethernet cable from the home router (or wherever the AP was connected for initial setup).
2. Connect the AP's uplink ethernet port to eth1 on the Pi (the USB-to-Ethernet adapter designated as the trunk interface)
      1. On APs that were originally consumer routers, there may be both a WAN/Internet port and LAN ports. Always connect to a LAN port. On purpose-built APs, there is typically a single ethernet port which serves as the uplink.
   
3. Verify the link is up on Pi:

        ip link show eth1

State must change from NO-CARRIER to UP. If it shows DOWN, check the cable and confirm the AP is powered on.

4. Verify the subinterfaces are up:

        ip addr show eth1.10
        ip addr show eth1.20

Both must show their correct IPs (192.168.10.1/24 and 192.168.20.1/24). If they show LOWERLAYERDOWN, eth1 does not have a physical link, check cable connections.