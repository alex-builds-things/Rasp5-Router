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
- Step 1 - Flash Raspberry Pi OS to SD card
- Step 2 - Configure network interfaces and IP Forwarding
- Step 3 -  Install and configure dnsmasq and Set up firewall rules
- Step 4 -  Test routing and internet access


## Understanding the Network's Design
This project uses 802.1Q VLAN tagging on a trunk port. A standard ethernet cables carries one network, while a trunk port carries multiple networks simultaneously by tagging each packet with a VLAN ID number. The AP (EAO670) receives packets from clients on each VLAN that was created, stamps each with their respective vlan id, and sends all traffic up a single ethernet cable to the Pi. The Pi reads the VLAN tag and routes each packet to the correct virtual interface.

# IP Address Plan

Interface         Role              IP Address                        Devices Served
  eth0            WAN               Obtained from ISP                 None
  eth1            Trunk Parent      No IP - Carries tagged traffic    Parent for eth1.10 and eth1.20
  eth1.10         VLAN10 gateway    192.168.10.1/24                   192.168.10.100-200
  eth1.20         VLAN20 gateway    192.168.20.1/24                   192.168.20.100-200
  eth2            Reserved          No IP configured                  Future use

| Interface| Role     | IP Address|  Devices Served  |
|----------|----------|-----------|------------|
| eth0 | WAN  | Obtained from ISP    |  None      |
| Cell 4   | Cell 5   | Cell 6    |  Cell      |
| Cell 1   | Cell 2   | Cell 3    |  Cell      |
| Cell 4   | Cell 5   | Cell 6    |  Cell      |

# Traffic Rules

  From              To                Decision
  VLAN 10           Internet          Allowed
  VLAN 20           Internet          Allowed
  VLAN 10           VLAN 20           Allowed
  VLAN 20           VLAN 10           Blocked


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


2 - Lost SSH access to Raspberry Pi after setting up GitHub Authentication:
    - While setting up SSH authentication for GitHub, I ran the following command 
      without specifying a filename:
      
        ssh-keygen -t ed25519 -C "email@example.com"
        
    - This overwrote the existing SSH key that was being used to connect to the 
      Raspberry Pi, as both were saved to the same default file (id_ed25519). 
      After this, attempting to SSH into the Pi returned the following error:
      
        pi@[assigned_IP_Address] Permission denied (publickey, password)

    - Resolution:
        STEP 1 - Attempted to access the Pi's SSH config directly via the SD card
            - The SD card was removed from the Pi and inserted into a Windows machine
            - The rootfs partition (where sshd_config lives) is Linux formatted (ext4) 
              and could not be read natively on Windows
            - DiskInternals Linux Reader was used to access the partition, however 
              writing back to the partition requires the paid version

        STEP 2 - Decided to reflash the SD card
            - Raspberry Pi Imager was used to flash a fresh Raspberry Pi OS lite (64-bit) image
            - SSH, username, and password were pre-configured inside the Imager settings 
              before writing, removing the need for a keyboard to set up the Pi

        STEP 3 - Restructured SSH keys to prevent this from happening again
            - A dedicated key was generated for GitHub:
                ssh-keygen -t ed25519 -C "email@example.com" -f ~/.ssh/id_ed25519_github
                
            - A dedicated key was generated for the Raspberry Pi:
                ssh-keygen -t ed25519 -C "pi-access" -f ~/.ssh/id_ed25519_pi
                
            - An SSH config file (~/.ssh/config) was created to map each key to its 
              respective host, ensuring the keys never conflict:

                # GitHub
                Host github.com
                  HostName github.com
                  User git
                  IdentityFile ~/.ssh/id_ed25519_github

                # Raspberry Pi
                Host rasp5-router.lan
                  HostName rasp5-router.lan
                  User piuser
                  IdentityFile ~/.ssh/id_ed25519_pi

    - Key lesson: Always use the -f flag when generating SSH keys to specify a filename. 
      Never run ssh-keygen without it if existing keys are present on the machine.