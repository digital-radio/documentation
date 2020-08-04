HOTSPOT AND WIFI CLIENT
===

[Tutorial used in below version](https://www.raspberrypi.org/forums/viewtopic.php?t=196263&fbclid=IwAR1Wz4QT-x2OJBdhlziibheB4ZyciOb5yxFsPnDVhzcCwVqYDwimK8FybXw)

[Tutorial that failed](https://blog.ithasu.org/2017/11/raspberry-pi-0w-as-both-wifi-client-and-access-point/?fbclid=IwAR15ptnzc2LExEk-qiOxTpmZSp3g0xqpRBntMLYzXPo-Xcih2m8e3g-cg4Q)

### Select IP network that Raspberry Pi is going to manage
192.168.50.0/24 (if available)

### Install software

Install hostapd so that Raspberry Pi can work as an access point
```bash 
sudo apt install hostapd

```

Install dnsmasq so that Raspberry Pi can provide network management services (DNS, DHCP) to wireless clients
```bash 
sudo apt install dnsmasq
```

### Configure the DHCP and DNS services for the wireless network

Rename the default configuration file and edit a new one:
```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo touch /etc/dnsmasq.conf
```

Add the following to the file and save it - use IP network chosen at first step:
```
interface=lo,uap0  # Listening interface
no-dhcp-interface=lo,wlan0
bind-interfaces
server=8.8.8.8
domain-needed
bogus-priv
dhcp-range=192.168.50.50,192.168.50.150,12h  # Pool of IP addresses served via DHCP
domain=uap0    # Local wireless DNS domain
address=/raspberrypi.uap0/192.168.50.1    # Alias for this router
```
Edit /etc/dhcpcd.conf to enable ssh in thru AP's ip instead of wifi client's ip - ssh using pi@raspberrypi.local is possible then.
```
interface uap0
```
Tell where the defaul config is. Add the following to the file /etc/default/hostapd
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
### Create start script in /usr/local/bin
```
#!/bin/bash

service hostapd stop
service dnsmasq stop

# Create the virtual device
/sbin/iw dev wlan0 interface add uap0 type __ap
# Uncomment the three following lines to get access to internet
sysctl net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -s 192.168.50.0/24 ! -d 192.168.50.0/24 -j MASQUERADE
iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE

# Fetch current wifi channel
CHANNEL=`iwlist wlan0 channel | grep Current | sed 's/.*Channel \([0-9]*\).*/\1/g'`
if [ -z "$CHANNEL" ]; then
    # Fetch first wifi channel
    CHANNEL=`iwlist wlan0 channel | grep Channel | head -n 1 | sed 's/.*Channel \([0-9]*\).*/\1/g'`
fi
export CHANNEL

# Create the config for hostapd
cat /etc/hostapd/hostapd.conf.tmpl | envsubst > /etc/hostapd/hostapd.conf

ifdown wlan0
ip link set dev uap0 up
ip addr add 192.168.50.1/24 broadcast 192.168.50.255 dev uap0

sleep 1

service hostapd restart
ifup wlan0
service dnsmasq restart
```

### Start with system
Insert the following line into /etc/rc.local before the exit 0
```
/bin/bash /usr/local/bin/hostapdstart_wifi
```

### SSH into raspberry
If you are connected to wifi Malina
```
ssh pi@192.168.50.1
or
ssh pi@raspberrypi.uap0
or
ssh pi@raspberrypi.local
```

If you are using your usb connection
```
ssh pi@raspberrypi.local
```
