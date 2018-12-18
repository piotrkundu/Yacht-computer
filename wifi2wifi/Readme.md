
# A wifi 2 wifi gateway using a Raspberry Pi and a USB wifi

A wifi 2 wifi gateway using a Raspberry Pi and a USB wifi dongle or a
USB high gain antenna.  Written to provide wifi onboard a yacht when
wifi us only availale at a distance where only a high gain wifi
antenna will get acceptable connection. Quite often only a good antenna
can make a good connection to the marina or a café. It does provide
local ip-numbers (192.168.1.xx) using a local dhcp and network address
translation (NAT) to forward the traffic to and from the external network
where the USB wifi is seen as a client. Hence the wifi2wifi label.

## Step by step instructions.

It should not be too hard to set up a Raspberry Pi to become a wifi
to wifi gateway.
My distribution is *Raspbian GNU/Linux 9*, *Linux raspberrypi 4.14.79-v7+ #1159*.
I have used nano as an editor, since emacs will take up too much space on the
limited micro-sd card, more on cleaning up unwanted packages below. 

Just replacing the files provided should work, but make a backup of your files before
replacing them with the provided ones. It's only tested on this distribution.

### Clean up and start :
```
apt update
apt upgrade
apt autoremove 
reboot
apt install dnsmasq hostapd
```
Stop services while we are updating.
```
systemctl stop hostapd
systemctl stop dnsmasq
```

Edit the files below, use the files in this repository as hints or just
copy and replace the original. Remember to make a copy of the original
ones, your files might not be identical.

```
nano /etc/dhcpcd.conf
mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
nano /etc/dnsmasq.conf
mkdir -p /etc/hostapd/
nano /etc/hostapd/hostapd.conf
nano /etc/default/hostapd
```
Only the line with net.ipv4.ip_forward=1 need updating, removing the comment mark. 
```
nano /etc/sysctl.conf
```
copy in :
```
/etc/iptables.ipv4.nat
```
If you want to prepare this youself the commands are :
```
iptables -t nat -A POSTROUTING -o wlan1 -j MASQUERADE
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan0 -o wlan1 -j ACCEPT
sh -c "iptables-save > /etc/iptables.ipv4.nat"
```
Next step is to update the interfaces file. It is in this file where wlan0 and wlan1
are described, wlan0 is the internal wifi interaface (server, dhcp server local) and
wlan1 is the external wifi client which obtain an ip address (dhcp client request)
from the hotspot wifi provider ashore.
```
nano /etc/network/interfaces
```
Remember to replace MAC address with your Raspberry Pi internal wlan MAC address,
```
hwaddress ether b8:27:eb:05:89:ec.
```
Edit
```
nano /etc/rc.local
```
the extra lines I included. The dhcp service refuse to start when there are both a client and a server 
specifies in the /etc/network/interfaces.


Dhcp server do not start anyway, it finds wlan1 being a dhcp client, and fails du start.
There is a test in the startup script. 
```
update-rc.d dhcpcd disable
```
Hence the manual dhcp start in rc.local, this will not restart automatically  if the
dhcp server fails or stops for some reason. It will need to be restared manually.


## Some extra commands that can be handy:

Changing the IP tables at the comand line :
```
iptables -t nat -A POSTROUTING -o wlan1 -j MASQUERADE
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan0 -o wlan1 -j ACCEPT
```
Copy the setup to a file and reading the setup from a file: 
```
sh -c "iptables-save > /etc/iptables.ipv4.nat"
iptables-restore < /etc/iptables.ipv4.nat
```
### Routing need to be fixed and here are some commands that can be useful.
```
route del -net 0.0.0.0 gw 192.168.1.1 netmask 0.0.0.0 dev eth0
route del -net 192.168.1.0  gw 0.0.0.0 netmask 255.255.255.0 dev eth0
route del -net 192.168.0.0  gw 0.0.0.0 netmask 255.255.255.0 dev wlan0
route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    304    0        0 wlan1
192.168.1.0     0.0.0.0         255.255.255.0   U     304    0        0 wlan1
```
Test the route with a ping command like :
```
ping vg.no
```

### Clean up a lot of things not needed :
```
apt remove   bridge-utils  wpagui  claws-mail libreoffice wolfram wolfram-engine\
smartsim   openoffice.org   openoffice   libreoffice  minecraft-pi\
libreoffice-common libreoffice-core libreoffice-gtk libreoffice-gtk2 libreoffice-java-common\
libreoffice-sdbc-hsqldb libreoffice-style-galaxy libreoffice-systray 

apt autoremove
```



