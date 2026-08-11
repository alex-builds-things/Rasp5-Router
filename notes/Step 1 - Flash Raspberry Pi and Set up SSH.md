# STEP 1 - Flashing Raspberry Pi and Set up SSH

## Part A - Installing Pi Operating System

## 1.1 - Which OS to use
Install Raspberry PI OS Lite (64 - bit).  The lite image uses command line which has faster response times, utilizes less memory and has a smaller attack surface.

## 1.2 Flash the OS with Raspberry Pi Imager
    1. Download and install Raspberry Pi Imager from: https://www.raspberrypi.com/software/
    2. Insert your MicroSD card into your computer
    3. Open Raspberry Pi Imager. Click CHOOSE Device and select Raspberry Pi 5.
    4. Click CHOOSE OS. Navigate to: Raspberry PI OS (other), Select Raspberry Pi OS Lite (64-bit).
    5. Click CHOOSE Storage and select your MicroSD card. ENSURE that the storage device selected is your microSD card. This as the selected drive will be permanently erased.
    6. Click NEXT, then navigate to the customization panel.

## 1.3 Configure OS Settings before flashing
    - Set hostname (You can assign you own hostname)
    - Set a username - Do not use the old default username "pi" - Write it down
    - Set a strong password - Write it down
    - Leave Wi-Fi fields blank - unless you want your Pi to connect to your home network via wifi.
    - Set time zone and keyboard layout
  
    - Check: Enable SSH
    - Slect: Use password authentication

    7. Click SAVE. Click YES to confirm writing. Click YES on the erase confirmation.
    8. When flashing is complete, safely eject the MicroSD card.


## Part B - First Boot and SSH Access

## 2.1 Physical Setup for Initial Configuration
The Pi will connect to your existing home network so you can SSH in and download and update packages. The Pi will be behind your existing firewall during this phase and will not be exposed to the internet directly.

    1. Insert the MicroSD card into the Pi 5.
    2. Plug both USB-to-Ethernet adapters into the blue USB 3.0 ports on the Pi.
    3. Connect the the Pi's built-in Etherport (eth0) to a LAN port on your home router using an ethernet cable (Do not connet eth1 or eth2 yet).
    4. Connect the power supply and power on the Pi. Wait approximately 90 seconds as the Pi expands its filesystem.

## 2.2 Find the Pi's IP Address
The Pi will receive an IP address from your ISP automatically. Find the Pi's IP addressing by accessing your home router's admin portal. You can also try accessing the Pi through the command line or terminal directly using the command: 'ssh (pi_username)@(pi_hostname).local' or 'ssh (pi_username)@(pi_hostname).lan' or 'ssh (pi_username)@192.168.X.X'

- NOTE: Pi-Username and Pi_hostname were both created in Part A above.

## 2.3 Connect via SSH
    5. Open Command Line or Terminal on your computer system.
    6. Type the following, replace the IP with your Pi's actual IP:
        ssh (pi_username)@192.168.X.X' or 'ssh (pi_username)@(pi_hostname).local' or 'ssh (pi_username)@(pi_hostname).lan'
    7. Type yes when asked about the fingerprint. Enter your password when prompted. Note that the cursor does not move while typing a password.
    8. A successful login window shows:
        (pi_username)@(pi_hostname):~$ or similar
   
- NOTE: Pi-Username and Pi_hostname were both created in Part A above
  
## Update the system
Never skip this critical step on a fresh install. An outdated system has known vulnerabilities and publicly documented vulnerabilities.

    1. Update the package list and install all updates:
         Enter the follwoing and press enter: sudo apt update
         Then: sudo apt full-upgrade -y
   
    2. Remove unused packages:
        sudo apt autoremove -y

    3. Reboot the Pi:
        sudo reboot


## Part C - Securing SSH Access

## 3.1 Verify SSH Configuration
1. Check if SSH is active:
    sudo systemctl status ssh
    - Must show: Active
   
2. Check critical SSH settings:
   sudo grep -E "PasswordAuthentication|PermitRootLogin" /etc/ssh/sshd_config
    - You should see:
       PasswordAuthentication yes
       PermitRootLogin no

3. If PermitRootLogin shows yes, fix it:
   1. sudo nano /etc/ssh/sshd_config

   2. Find "PermitRootLogin" and change it to:
      1. PermitRootLogin no
   
   3. Save: Ctrl+X, Press Y, Press Enter
   
   4. Restart SSH:
      sudo systemctl restart ssh


## 3.2 SSH Key Authentication
   