## Install and configure dnsmasq and Set up firewall rules

# What is happening here
dnsmasq handles two jobs simultaneously: DHCP and DNS. One dnsmasq instance serves both VLANs from separate address pools.

# 1. Install dnsmasq
1.1 Install dnsmasq
    
        sudo apt install dnsmasq -y

1.2 Stop dnsmasq and configure it:
    
        sudo systemctl stop dnsmasq

# 2. Resolve Port 53 Conflicts
