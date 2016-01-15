#Setting-up the RPi as a WIFI access point

This tutorial is mostly adapted from [the adafruit website](https://learn.adafruit.com/downloads/pdf/setting-up-a-raspberry-pi-as-a-wifi-access-point.pdf)

* Hardware used: [Plugable USB 2.0 Wireless N 802.11n 150 Mbps Nano WiFi Network Adapter (Realtek RTL8188CUS Chipset)](http://www.amazon.co.uk/gp/product/B00H28H8DU?psc=1&redirect=true&ref_=oh_aui_search_detailpage)

### Installations
First, install the HostAPD (Host access point daemon) and the DHCP server:
* hostapd is a user space daemon for wireless access point and authentication servers.
* Internet Systems Consortium's, Dynamic Host Configuration Protocol server for automatic IP address assignment


    sudo apt-get install hostapd isc-dhcp-server
    
### /etc/dhcp/dhcpd.conf
> edit the file that sets up our DHCP server: - this allows wifi connections to automatically get IP addresses, DNS, etc.


    sudo nano /etc/dhcp/dhcpd.conf

> comment out the lines: #option domain-name "example.org";
#option domain-name-servers ns1.example.org, ns2.example.org;


> uncomment authoritative; after "If this DHCP server is the official DHCP server for the local network, the authoritative directive should be uncommented"

Then at the bottom of the file add the config you want:

    subnet 192.168.42.0 netmask 255.255.255.0 {
    range 192.168.42.10 192.168.42.90;
    option broadcast-address 192.168.42.255;
    option routers 192.168.42.1;
    default-lease-time 600;
    max-lease-time 7200;
    option domain-name "local";
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    }

### /etc/default/isc-dhcp-server

Add WLAN0 as the interface: 

    INTERFACES="wlan0"
    
In case wlan0 is being used, run: 

    sudo ifdown wlan0
    
### /etc/network/interfaces

> Next we will set up the wlan0 connection to be static and incoming

instead of: 

    auto wlan0
    allow-hotplug wlan0
    iface wlan0 inet manual
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

write: 

    #auto wlan0
    
    allow-hotplug wlan0
    
    iface wlan0 inet static
    address 192.168.42.1
    netmask 255.255.255.0
    
    #iface wlan0 inet manual
    #wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

then run: 

    sudo ifconfig wlan0 192.168.42.1

### /etc/hostapd/hostapd.conf
Create a new file configuring the access points details

    interface=wlan0
    #driver=rtl871xdrv
    ssid=slate2learn
    hw_mode=g
    channel=6
    macaddr_acl=0
    auth_algs=1
    ignore_broadcast_ssid=0
    wpa=2
    wpa_passphrase=clementine
    wpa_key_mgmt=WPA-PSK
    wpa_pairwise=TKIP
    rsn_pairwise=CCMP
    
Note that we uncomment the driver line here (as compared to adafruit instructions).

### /etc/default/hostapd
Tell the pi to find this configuration file by editing to: 

    DAEMON_CONF="/etc/hostapd/hostapd.conf"
    
Don't forget to uncomment the line!
    
### Final steps
Now to test is it works: 

    sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf

To set it up as a deamon service: 

    sudo service hostapd start
    sudo service isc-dhcp-server start
    
To make sure it runs every time on boot: 

    sudo update-rc.d hostapd enable
    sudo update-rc.d isc-dhcp-server enable

