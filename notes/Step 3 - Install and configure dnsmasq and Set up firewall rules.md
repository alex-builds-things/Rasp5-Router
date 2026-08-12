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