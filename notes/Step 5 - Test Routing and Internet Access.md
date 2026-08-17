# Test Routing and Internet Access


# 1 - Test DHCP
Connect a device to the SSID created and ensure that it receives an 192.168.10.X IP address automatically

# 2 - Test Internet 
        ping eth1 1.1.1.1 -c 4
        ping google.com -c 4

# 3 - Reboot Survival
    #3.1 Reboot
        sudo reboot

    #3.2 Run
        ip addr show
        sudo systemctl status dnsmasq
        sudo systemctl status nftables
        sudo nft list ruleset | cat
        ping 1.1.1.1 -c 4

    Everything must come back exactly as before reboot.


# 4 - Perfomance Tuning
    #4.1 Increase Network buffers
    Open sysctl.conf
            sudo nano /etc/sysctl.conf

    Add the following:
            net.core.rmem_max=16777216
            net.core.wmem_max=16777216
            net.core.netdev_max_backlog=5000
            net.ipv4.tcp_rmem=4096 87380 16777216
            net.ipv4.tcp_wmem=4096 65536 16777216
            net.core.default_qdisc=fq
            net.ipv4.rcp_congestion_controller=bbr

    Ctrl + X, Press Y, Press Enter

    Apply:
            sudo sysctl -p


    #4.2 Hardware Offloading
            sudo ethtool -K eth0 rx on tx on
            sudo ethtool -K eth1 rx on tx on

        Note: USB ethernet adapters often do not support hardware offloading. If ethtool returns an error, the adapter does not supprt it. This is not a problem.

    #4.3 Monitor Bandwidth
    Install vnstat:
            sudo apt install vnstat -y
            sudo systemctl enable vnstat
            vnstat -l -i eth0
            vnstat -l -i eth1

            