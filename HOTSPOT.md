HOTSPOT
===

[Tutorial](https://www.raspberrypi.org/documentation/configuration/wireless/access-point-routed.md?fbclid=IwAR2KwNiKAsOHfvXIV6imXZ7uxa81u91UsF4i4v-bpGNRhtXSWA4yf4uIq7k)

### Select IP network that Raspberry Pi is going to manage
192.168.4.0/24 (if available)

### Install hostapd so that Raspberry Pi can work as an access point
```bash 
sudo apt install hostapd

```
### Enable the wireless access point service and set it to start when your Raspberry Pi boots
```bash 
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
```

### Install software

Install dnsmasq so that Raspberry Pi can provide network management services (DNS, DHCP) to wireless clients
```bash 
sudo apt install dnsmasq
```

Install netfilter-persistent and iptables-persistent to save firewall rules
```bash 
sudo DEBIAN_FRONTEND=noninteractive apt install -y netfilter-persistent iptables-persistent
```

### Define the wireless interface IP configuration

Go to the end of the file 
```
/etc/dhcpcd.conf
```

and add the following (static IP for Raspberry Pi) - use IP network chosen at first step:
```
interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
```

### Configure the DHCP and DNS services for the wireless network

Rename the default configuration file and edit a new one:
```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo touch /etc/dnsmasq.conf
```

Add the following to the file and save it - use IP network chosen at first step:
```
interface=wlan0    # Listening interface
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h    # Pool of IP addresses served via DHCP
domain=wlan    # Local wireless DNS domain
address=/gw.wlan/192.168.4.1    # Alias for this router
```
The Raspberry Pi will deliver IP addresses between 192.168.4.2 and 192.168.4.20, with a lease time of 24 hours, to wireless DHCP clients. You should be able to reach the Raspberry Pi under the name gw.wlan from wireless clients.

### Ensure WiFi radio is not blocked on your Raspberry Pi
```bash 
sudo rfkill unblock wlan
```

### Configure the access point software

Create file
```bash 
sudo touch /etc/hostapd/hostapd.conf
```

Add the following to the file (set your own wpa_passphrase and if needed ssid and country_code):
```
country_code=PLN
interface=wlan0
ssid=Malina
hw_mode=g
channel=7
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=MojeHaslo
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

### Run your new wireless access point

Restart your Raspberry Pi
```bash
sudo systemctl reboot
```

If you want to connect via ssh
```
Connect to wifi Malina
```
```bash 
ssh pi@192.168.4.1
```
or 
```bash 
ssh pi@gw.wlan
```
