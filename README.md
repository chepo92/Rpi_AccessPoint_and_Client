# Status: working 
As April 8 2019 this is working fine with raspbian jessie and stretch. Let me know if you have any issue

# Raspberry Pi as AccessPoint and Client at the same time! 
Rpi3B configuration to use it as WiFi AP and Client simultaneously on the same onboard adapter of the Pi, this also allows the pi to share its internet conection (as a repeater), you can connect to your RPi without the need to be conected to a router nor ethernet, and operate "headless" or "WirelessHead" with the use of ssh or VNC server, having access to the desktop with a tablet or phone with VNC connect

## Configure WiFi client
create a copy of /etc/wpa_supplicant/wpa_supplicant.conf  in the same folder with the name for your interface e.g interface wlan0 wpa_supplicant-wlan0.conf and set in it the ssid and pass of your router

## Install hostapd and dnsmasq
in shell: 
```
sudo apt-get update
sudo apt-get install hostapd dnsmasq
```

## Set interfaces wlan0 and uap0 
In file /etc/network/interfaces
Comment all lines except: source-directory /etc/network/interfaces.d
Eg. 

```
source-directory /etc/network/interfaces.d
#auto lo
#iface lo inet loopback

#iface eth0 inet manual

#auto wlan0
#allow-hotplug wlan0
#iface wlan0 inet dhcp
#   wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

#allow-hotplug uap0
#auto uap0
```

or

```
source-directory /etc/network/interfaces.d
auto lo
auto eth0
auto wlan0

iface lo inet loopback
#iface eth0 inet manual

allow-hotplug wlan0
iface wlan0 inet manual
   wpa-conf /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```


## Configure static ip in uap0 
E.g. Content of /etc/dhcpcd.conf
```
interface uap0
 static ip_address=192.168.50.1/24
```

## Configure HostAP
Set the name, channel and pass for your raspberry hotspot
E.g Content of /etc/hostapd/hostapd.conf

```
interface=uap0
#driver=nl80211
ssid=PiAP
hw_mode=g
channel=yourAPChannel
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=yourPassword
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```


** Set the SSID and password you want the default would be "PiAP" and "yourPassword" respectively, also the channel must be the same to the one you will be connecting the RPi to. (i.e the channel must be the same as your router's)

## Configure the daemon for the host AP
Add the following to /etc/default/hostapd

`DAEMON_CONF="/etc/hostapd/hostapd.conf"`

## Configure AP dns
Content of /etc/dnsmasq.conf
```
interface=lo,uap0
dhcp-range=192.168.50.50,192.168.50.150,12h 
```

## Configure forwarding
Uncomment the next line in /etc/sysctl.conf

`net.ipv4.ip_forward=1`


## Configure Start of services
Add to /etc/rc.local 
```
service hostapd stop
service dnsmasq stop
service dhcpcd stop
iw dev wlan0 interface add uap0 type __ap
iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
service hostapd start
service dnsmasq start
service dhcpcd start
```

## Disable auto start of services
In shell: 
```
sudo systemctl stop hostapd &&
sudo systemctl stop dnsmasq &&
sudo systemctl stop dhcpcd &&
sudo systemctl disable hostapd &&
sudo systemctl disable dnsmasq &&
sudo systemctl disable dhcpcd &&
sudo systemctl start hostapd &&
sudo systemctl start dnsmasq &&
sudo systemctl start dhcpcd
```

## Reboot
`sudo reboot`

If everything is configured correctly your Pi will be connected to your wifi and also you will see its hotspot network (e.g. PiAP), you will be able to connect to it via any of its two ip (client or hot spot)

## Troubleshoot
### Problem: hostapd service is masked 
`sudo systemctl unmask hostapd.service`

### Problem: Not conecting to router

Diagnostic command `service dhcpcd status`

Throws: uap0: IAID conflicts with one assigned to wlan0

Fix:  `sudo update-rc.d -f dhcpcd remove`

