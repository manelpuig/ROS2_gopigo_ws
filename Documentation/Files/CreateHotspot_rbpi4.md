Configuring a Wi-Fi hotspot on a Raspberry Pi 4 running Ubuntu 22.04 can be done through the command line using `hostapd` and `dnsmasq`. Here's a step-by-step guide to set up a Wi-Fi hotspot:

1. **Prerequisites:**
   - Make sure you have a Raspberry Pi 4 with Ubuntu 22.04 installed and a compatible Wi-Fi adapter.
   - You should be connected to the internet via Ethernet or another Wi-Fi connection.

2. **Update and Upgrade:**
   Update your package list and upgrade your system to ensure you have the latest packages:

   ```bash
   sudo apt update
   sudo apt upgrade
   ```

3. **Install Hostapd and Dnsmasq:**
   Install the necessary software for creating a Wi-Fi hotspot:

   ```bash
   sudo apt install hostapd dnsmasq
   ```

4. **Configure a Static IP Address:**
   You need to configure a static IP address for the Wi-Fi interface (e.g., `wlan0`). Edit the network configuration file:

   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```

   Add the following lines to configure a static IP for the `wlan0` interface. Replace the placeholders with your network details:

   ```yaml
   wifis:
       wlan0:
           addresses: [192.168.1.1/24]
           dhcp4: no
           optional: true
           access-points:
               "Your-SSID-Name":
                   password: "Your-Password"
   ```

   Save and exit the text editor.

5. **Set Up Hostapd Configuration:**
   Create or modify the `hostapd` configuration file:

   ```bash
   sudo nano /etc/hostapd/hostapd.conf
   ```

   Add the following configuration:

   ```plaintext
   interface=wlan0
   driver=nl80211
   ssid=Your-SSID-Name
   hw_mode=g
   channel=6
   wmm_enabled=0
   macaddr_acl=0
   auth_algs=1
   ignore_broadcast_ssid=0
   wpa=2
   wpa_passphrase=Your-Password
   wpa_key_mgmt=WPA-PSK
   wpa_pairwise=TKIP
   rsn_pairwise=CCMP
   ```

   Save and exit the text editor.

6. **Configure Dnsmasq:**
   Configure the `dnsmasq` DHCP server by creating a configuration file:

   ```bash
   sudo nano /etc/dnsmasq.d/wlan0
   ```

   Add the following lines:

   ```plaintext
   interface=wlan0
   dhcp-range=192.168.1.2,192.168.1.20,255.255.255.0,24h
   domain=wlan
   address=/gw.wlan/192.168.1.1
   ```

   Save and exit.

7. **Enable IPv4 Forwarding:**
   Enable IPv4 packet forwarding by editing the sysctl configuration file:

   ```bash
   sudo nano /etc/sysctl.conf
   ```

   Uncomment the following line (remove the `#` at the beginning):

   ```plaintext
   net.ipv4.ip_forward=1
   ```

   Save and exit.

8. **Start the Services:**
   Start and enable `hostapd` and `dnsmasq`:

   ```bash
   sudo systemctl unmask hostapd
   sudo systemctl enable hostapd
   sudo systemctl start hostapd
   sudo systemctl enable dnsmasq
   sudo systemctl start dnsmasq
   ```

9. **Enable IP Forwarding and Masquerading:**
   Set up IP forwarding and masquerading to allow your Raspberry Pi to route traffic between clients and the internet:

   ```bash
   sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
   sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
   ```

   Make these rules persistent:

   ```bash
   sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
   ```

   Open the `rc.local` file for editing:

   ```bash
   sudo nano /etc/rc.local
   ```

   Add the following line just before the `exit 0` line to restore the iptables rules on boot:

   ```bash
   iptables-restore < /etc/iptables.ipv4.nat
   ```

   Save and exit.

10. **Reboot Your Raspberry Pi:**
    Reboot your Raspberry Pi for the changes to take effect:

    ```bash
    sudo reboot
    ```

Your Raspberry Pi should now be broadcasting a Wi-Fi hotspot with the SSID and password you specified in the `hostapd` configuration. Clients can connect to this hotspot, and the Raspberry Pi will provide internet access through its Ethernet connection.

Please replace "Your-SSID-Name" and "Your-Password" with your desired SSID and password for the hotspot. Additionally, ensure that your Raspberry Pi is compatible with creating a Wi-Fi hotspot and that you have a compatible Wi-Fi USB adapter if your built-in Wi-Fi doesn't support it.