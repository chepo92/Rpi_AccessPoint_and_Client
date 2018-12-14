# Rpi_AccessPoint_and_Client
Rpi3B config to use as WiFi AP and Client simultaneously on the same onboar 

## Configure WiFi client
open /etc/wpa_supplicant/wpa_supplicant.conf and set ssid and pass of your router

## Install hostapd and dnsmasq
in shell: 
sudo apt-get install hostapd dnsmasq

## Configure static ip in uap0 
Content of /etc/dhcpcd.conf

interface uap0
 static ip_address=192.168.50.1/24

## Configure HostAP
Content of /etc/hostapd/hostapd.conf

interface=uap0
#driver=nl80211
ssid=testAP
hw_mode=g
channel=1
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=1234567890
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

## Configure the daemon for the host AP
Add the following to /etc/default/hostapd

DAEMON_CONF="/etc/hostapd/hostapd.conf"

## Configure AP dns
Content of /etc/dnsmasq.conf

interface=lo,uap0
dhcp-range=192.168.50.50,192.168.50.150,12h 

## Configure Startup
Add to /etc/rc.local 

service hostapd stop
service dnsmasq stop
service dhcpcd stop
iw dev wlan0 interface add uap0 type __ap
iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
service hostapd start
service dnsmasq start
service dhcpcd start


## Reboot
