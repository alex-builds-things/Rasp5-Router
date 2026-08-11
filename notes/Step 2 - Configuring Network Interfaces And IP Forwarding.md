## Configuring Network Interfaces - trunk and Sub-interfaces

# What is happening here
In this step eth1 will be configuraged as a trunk a parent, it will be the physical interface with no IP address of its own. The two virtual subinterfaces (VLAN10 - 192.168.10.1/24, VLAN20 - 192.168.20.1/24) that will be created will be configured on top of eth1.

The Pi sends and receives tagged traffic on eth1, reads the VLAN ID, and routes each packet to the correct subinterface. eth0 will get its IP automatically when this entire project is complete.


# 1. Configure eth0 - WAN interface
# 1.1 Create a NetworkManager profile for the WAN port:
    $ sudo nmcli connection add\
        type ethernet \
        ifname eth0 \
        con-name wan \
        ipv4.method auto \
        ipv6.method disabled

    # Turn on Connection:
        sudo nmcli connection up wan


# 2. Configure eth1 - Trunk Parent Interface
Eth1 carries tagged traffic for both VLANs.

2.1 Create trunk parent profile with no IP
    $ sudo nmcli connection add\
        type ethernet \
        ifname eth1 \
        con-name trunk-eth1\
        ipv4.method disbaled \
        ipv6.method disabled

    # Turn on Connection:
        sudo nmcli connection up trunk-eth1


# 3. Configure eth1.10 - VLAN 10 subinterface, 192.168.10.1
3.1 Create vlan 10 interface
    $ sudo nmcli connection add\
        type vlan \
        con-name vlan10 \
        ifname eth1.10 \
        dev eth1 \
        id 10 \
        ipv4.method manual \
        ipv4.addresses 192.168.10.1/24 \
        ipv4.never-default yes \
        ipv6.method disabled        

    # Turn on Connection:
        sudo nmcli connection up vlan10



# 4. Configure eth1.20 - VLAN 20 subinterface, 192.168.20.1
4.1 Create vlan 20 interface
    $ sudo nmcli connection add\
        type vlan \
        con-name vlan20 \
        ifname eth1.20 \
        dev eth1 \
        id 10 \
        ipv4.method manual \
        ipv4.addresses 192.168.20.1/24 \
        ipv4.never-default yes \
        ipv6.method disabled        

    # Turn on Connection:
        sudo nmcli connection up vlan20


# 5. Reserve eth2 interface
eth2 is physically connected but intentionally unconfigured. Creating a disabled profile documents its reserved status and prevents NetworkManager from trying to auto-configure it.

5.1 Create a disabled profile for eth2
    $ sudo nmcli connection add\
        type ethernet \
        ifname eth2 \
        con-name eth2-reserved \
        ipv4.method disabled \
        ipv6.method disabled \    


# 6. Verify configurations and default route
$ ip addr show

You must see:
    eth0:       inet [Temporary IP from ISP during configuration]
    eth1:       no inet line [Trunk parent has no IP]
    eth1.10:    inet 192.168.10.1/24
    eth1.20:    inet 192.168.20.1/24
    eth2:       no inet line [Reserved]

Verify default route
    $ ip route show

the 'default via' line must reference eth0. If 1.10 or 1.20 appears as default routes, the ipv4.never-default flag was not applied. Re-run Steps 2 - 4, paying keen attention to the never-default flag.



## Enabling IP Forwarding

# What is happening here
Linux discards packets that arrive on one interface but are destined for a different network by default. IP fordwaring tells the kernel to pass those packets through fpr routing. Without it, no traffic moves between VLANs or reaches the internet regardless of any other configuration

# 1. Open kernel parameters file:
    $ sudo nano /etc/sysctl.conf

# 2. Find and replace this line (If line is present, remove the "#" sign):
    #net.ipv4.ip_forward=1 

    net.ipv4.ip_forward=1

# 3. Save: Ctrl+X, Press Y on keyboard, Press Enter. Apply immediately:
    $ sudo sysctl -p

# 4. Verify output
    $ cat /proc/sys/net/ipv4/ip_forward

    Must output: 1. If it shows 0, repeat Steps 1 - 3


