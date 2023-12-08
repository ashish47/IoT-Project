## Project Demo Video Links
- Video 1 - https://photos.app.goo.gl/bf7bgG62m9yhUfPJ6
- Video 2 - https://photos.app.goo.gl/ZNfb9WHFfuGQ5YsY9

## To set up your own RPi Hotspot -

Note - You can just copy files in this repo if you don't have or plan to make other changes to Networking. 
Following the below steps exactly is the best way to get the hotspot On.

### Flash Raspian OS on an SD card usiing RPi Imager - https://www.raspberrypi.com/software/

### Set Static Address
Copy below code and paste at the bottom of /etc/dhcpcd.conf file
```
denyinterfaces wlan0
```

Copy below code and paste at the bottom of /etc/network/interfaces file
auto lo
```
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

allow-hotplug wlan0
iface wlan0 inet static
    address 192.168.5.1
    netmask 255.255.255.0
    network 192.168.5.0
    broadcast 192.168.5.255
```
### Configure Hostapd
Copy below code and paste at the bottom of /etc/hostapd/hostapd.conf file. (Hotspot name is variable, ex - IoT-Project like below)
```
interface=wlan0
driver=nl80211
ssid=IoT-Project
hw_mode=g
channel=6
ieee80211n=1
wmm_enabled=1
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_passphrase=raspberry
rsn_pairwise=CCMP
```
Copy below code and paste at the bottom of /etc/default/hostapd file
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
### Configure Dnsmasq
Copy below code and paste at the bottom of /etc/dnsmasq.conf file
```
interface=wlan0 
listen-address=192.168.5.1
bind-interfaces 
server=8.8.8.8
domain-needed
bogus-priv
dhcp-range=192.168.5.100,192.168.5.200,24h
```
### Forward Internet
Copy below code and paste at the bottom of /etc/sysctl.conf file
```
net.ipv4.ip_forward=1
```
Run the following commands-
```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```
Copy below code and paste at the bottom of /etc/rc.local file before the 'exit 0' line
```
iptables-restore < /etc/iptables.ipv4.nat 
```
### Reboot
```
sudo reboot
```
