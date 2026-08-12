## Install and configure dnsmasq and Set up firewall rules

# What is happening here
dnsmasq handles two jobs simultaneously: DHCP and DNS. One dnsmasq instance serves both VLANs from separate address pools.

# 1. Install dnsmasq
1.1 Install dnsmasq
    
        sudo apt install dnsmasq -y

1.2 Stop dnsmasq and configure it:
    
        sudo systemctl stop dnsmasq

# 2. Resolve Port 53 Conflicts
Pi OS sometimes runs systemd-resolved on port 53, which blocks dnsmasq from starting.

2.1 Check of systemd-resolved is active:
    
        sudo systemctl is-active systemd-resolved

2.2 If the output is "active", disable it:

        sudo systemctl disable systemd-resolved
        sudo systemctl stop systemd-resolved

2.3 Check if /etc/resolv.conf is a symlink:

    ls -la /etc/resolv.conf

If the output contains an arrow, it is a symlink. Remove it and create a plain file:

        sudo rm -f /etc/resolv.conf
        echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
        echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf


# 3. Back up the default configuration
Back up and replace the default dnsmasq configuration:

        sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.bak

# 4. Write the dnsmasq Configuration
1. Create a clean configuration file:
        
            sudo nano /etc/dnsmasq.conf

2. Paste the following: It excludes steps taken to resolve Challendge #3 - Unable to copy configuration files from Pi to Local System (found in readme file):
   
        #----- Global Settings --------------------------------
        # Listen on VLAN subinterfaces only - never on WAN port (th0)
        interface=eth1.10
        interface=eth1.20
        bind-interfaces

        # Upstream DNS: external queries forwarded to Cloudflare and Google
        server=1.1.1.1
        server=8.8.8.8

        # Cache 1000 DNS entries for faster repeat lookups
        cache-size=1000

        # Block DNS rebinding attacks
        stop-dns-rebind
        rebind-localhost-ok

        # ----- VLAN 10 (eth1.10) -------------------------
        # Assign IPs 192.168.10.100-200, 24-hour lease
        dhcp-range=eth1.10,192.168.10.100,192.168.10.200,255.255.255.0,24h

        # Tell devices that the Pi is their gateway
        dhcp-option=eth1.10,option:router,192.168.10.1

        # Tell devices that the Pi handles their DNS
        dhcp-option=eth1.10,option:dns-server,192.168.10.1


        # ----- VLAN 20 (eth1.10) -------------------------
        # Assign IPs 192.168.20.100-200, 24-hour lease
        dhcp-range=eth1.20,192.168.20.100,192.168.20.200,255.255.255.0,24h

        # Tell devices that the Pi is their gateway
        dhcp-option=eth1.20,option:router,192.168.20.1

        # Tell devices that the Pi handles their DNS
        dhcp-option=eth1.20,option:dns-server,192.168.20.1


        # -------- Logging ----------------
        log-dhcp
3. Save: Ctrl + X, press Y, press enter
   
4. Start and enable dnsmasq
            sudo systemctl start dnsmasq
            sudo systemctl enable dnsmasq

5. Verify dnsmasq is running:
            sudo systemctl status dnsmasq

6. Must show: Active:active (running). If it shows failed:
            sudo journalctl -u dnsmasq 