# Raspberry Pi Router Project: Rasp5-Router

## What is this?
This is a home router router using a Raspberry Pi 5, following the PiDIYLAB tutorial. Link: https://pidiylab.com/how-to-build-raspberry-pi-router/

The tutorial is for the Raspberry Pi 4, so slight changes were made for the Pi 5. Additionally, I added vlan configuration to the project.

The main changes were:

# 1 - Replace dhcp with NetworkManager
PI OS bookworm is required for Pi 5 and replaces dhcp with NetworkManager. If this is not done, all static IP commands in the original tutorial silently fails, so dhcp was changed to nmcli.

# 2 - Replace iptables with nftables
PI OS Bookwork uses nftables as its firewall backend. iptables commands from the original tutorial will conflict with the OS. So all iptables commands were replaced with nftables commands.


## Hardware used
-   Raspberry Pi 5
-   Samsung Pro Endurance 32 GB MicroSD
-   Official Pi 5 27W USB-C PSU
-   Cable Matters USB 3.0 Gigabit USB-to-Ethernet Adapter
-   Xtech USB-to-Ethernet Adapter
-   Two Cat 5e ethernet cables
-   TP-Link EAP670 Wifi Access Point


## Steps to be completed:
- Flash Raspberry Pi OS to SD card
- Configure network interfaces
- Set up firewall rules
- Test routing and internet access


## Understanding the Network's Design
This project uses 802.1Q VLAN tagging on a trunk port. A standard ethernet cables carries one network, while a trunk port carries multiple networks simultaneously by tagging each packet with a VLAN ID number. The AP (EAO670) receives packets from clients on each VLAN that was created, stamps each with their respective vlan id, and sends all traffic up a single ethernet cable to the Pi. The Pi reads the VLAN tag and routes each packet to the correct virtual interface.


## Challenges and How they were resolved
1 - Inability to push to git rep:
    - When running the git push command in terminal, I was prompted for github username and password. After proviing credentials, an error message appeared saying:
    "Invalid username or token. Password authentication is not supported for git operations.."
    - I did not know this at the item so i had to research the solution. I found two options
        
        OPTION A - Personal Access Token
            - This is a quick fix and replaces your password with a token. However, tokens expire.

        OPTION B - SSH Key
            - This utilizes a SSH key that is generated on your device and copied to github. 

            - I chose this option because it does not expire, and it required me to switch my repo remote from HTTPS to SSH using the following "git remote set-url origin git@github.com:profilename/nameofrepo.git"