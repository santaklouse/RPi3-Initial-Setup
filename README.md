## RPi-Initial-Setup
### Image writing (Mac with Terminal.app)
#### Unmount SD Card and Write Raspbian Image
```
diskutil umount /dev/disk2s1
sudo dd if=~/Downloads/2017-07-05-raspbian-jessie-lite.img of=/dev/disk2 bs=1m
```
#### ENABLE UART
https://www.raspberrypi.org/documentation/configuration/uart.md
```
echo "enable_uart=1" > /Volumes/boot/config.txt
```
#### GPU MEMORY ALLOCATION
https://www.raspberrypi.org/documentation/configuration/config-txt/memory.md
```
echo "gpu_mem=16" >> /Volumes/boot/config.txt
```
#### Unmount SD Card and Remove it
```
diskutil umount /Volumes/boot
```

### OS INITIAL SETUP
```
sudo bash
```
#### DISABLE IPv6
https://www.raspberrypi.org/forums/viewtopic.php?t=138899
```
cat <<EOF > /etc/modprobe.d/ipv6.conf
alias net-pf-10 off
alias ipv6 off
options ipv6 disable_ipv6=1
blacklist ipv6
EOF

cat <<EOF >> /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
net.ipv6.conf.eth0.disable_ipv6 = 1
net.ipv6.conf.[interface].disable_ipv6 = 1
EOF
```
#### NETWORK CONFIGURATION
wlan0 = Internal WiFi Adaptor

wlan1 = USB WiFi Dongle (Using TP-Link TL-WN725N)
```
cat <<EOF > /etc/network/interfaces 
source-directory /etc/network/interfaces.d
auto lo
iface lo inet loopback
allow-hotplug wlan0
iface wlan0 inet static
address 192.168.100.1
netmask 255.255.255.0
allow-hotplug wlan1
iface wlan1 inet manual
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
EOF

cat <<EOF > /etc/wpa_supplicant/wpa_supplicant.conf
country=JP
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="SSID"                 # < Change as you need
    psk="PASSWORD"              # < Change as you need
}
EOF
```
#### ENABLE REQUIRED SERVICE
```
systemctl enable ssh
systemctl enable avahi-daemon
reboot
```

#### REMOVE UNNECESSARY PACKAGES AND UPDATE OS
```
ssh pi@raspberrypi.local
sudo bash

echo 'deb http://ftp.jaist.ac.jp/raspbian/ jessie main contrib non-free rpi' > /etc/apt/sources.list
apt-get update
apt-get autoremove --purge -y alsa-utils bluez cifs-utils dosfstools dphys-swapfile ed fbset kbd keyboard-configuration nano nfs-common ntp rsyslog vim-tiny v4l-utils xauth xdg-user-dirs xkb-data
apt-get autoremove --purge -y
apt-get -y upgrade
```
#### INSTALL REQUIRED PACKAGES
```
apt-get install -y vim ntpdate git
apt-get clean
```
#### TIME CONFIGURATION
```
ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
ntpdate ntp.nict.jp
cat <<EOF > /etc/cron.hourly/ntpdate
#!/bin/sh
ntpdate ntp.nict.jp >/dev/null 2>&1
EOF
chmod 755 /etc/cron.hourly/ntpdate
```
#### UPDATE FIRMWARE
```
rpi-update
```
#### FSTAB
```
rm -rf /tmp/*
rm -rf /var/tmp/*
rm -rf /var/log/*
echo 'tmpfs  /tmp           tmpfs defaults,size=32m,noatime,mode=1777 0 0' >> /etc/fstab
echo 'tmpfs  /var/tmp       tmpfs defaults,size=16m,noatime,mode=1777 0 0' >> /etc/fstab
echo 'tmpfs  /var/log       tmpfs defaults,size=16m,noatime,mode=0755 0 0' >> /etc/fstab
```
#### HOSTNAME CONFIGURATION
```
echo "broccoli" > /etc/hostname
echo "127.0.0.1       localhost" > /etc/hosts
echo "127.0.1.1       broccoli" >> /etc/hosts
```
#### Done
```
reboot
```
