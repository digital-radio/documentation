CONNECT WIFI
===

[Tutorial](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)

### List wireless networks
```bash
sudo iwlist wlan0 scan
```

### Connect with Wi-Fi network
Add the following lines
```
network={
    ssid="testing"
    psk="testingPassword"
}
```

To the following file
```
/etc/wpa_supplicant/wpa_supplicant.conf
```

Apply changes
```bash
wpa_cli -i wlan0 reconfigure
```

Check if connected
```bash
ifconfig wlan0
```
