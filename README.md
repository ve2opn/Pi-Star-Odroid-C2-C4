# Pi-star for ODROID-C2 and C4

**This is a modification of the famous pi-star DMR hotspot project:**  
https://www.pistar.uk/
https://github.com/AndyTaylorTweet

## Hat

![HAT](https://raw.githubusercontent.com/phl0/MMDVM_HS_Dual_Hat/master/mmdvm_hs_dual_hat.png | width=100)

HW: https://github.com/phl0/MMDVM_HS_Dual_Hat/blob/master/hardware/r1.3/mmdvm_hs_dual_hat.pdf

FW: https://github.com/juribeparada/MMDVM_HS

## Boards

![C2](https://dn.odroid.com/homebackup/201602/ODROID-C2.png | width=100)

https://wiki.odroid.com/odroid-c2/odroid-c2

![C4](https://wiki.odroid.com/_media/odroid-c4/c4_k.jpg | width=100)

https://wiki.odroid.com/odroid-c4/odroid-c4

## OS

C2: [ubuntu-20.04-3.16-minimal-odroid-c2-20210201.img.xz](https://odroid.in/ubuntu_20.04lts/c2/ubuntu-20.04-3.16-minimal-odroid-c2-20210201.img.xz)

C4: [ubuntu-22.04-4.9-minimal-odroid-c4-hc4-20220705.img.xz](https://odroid.in/ubuntu_22.04lts/C4_HC4/ubuntu-22.04-4.9-minimal-odroid-c4-hc4-20220705.img.xz)

# Differences

 - [Pi-star](https://www.pistar.uk/downloads/) project provides **armhf** images for Pi and ODROID-XU4. Unfortunately, these don't run directly on C2 and C4 boards, which use **aarch64 (arm64)** OS.
 - Ubuntu images for C2/C4 mount a partition to **/media/boot** not to **/boot** so many pi-star scripts need a modification
 - 40-pin connectors on C2 and C4 look similar to Pi but they are different - especially the pins for hat reset / boot.
 - GPIO# are different for each board i.e. GPIO2xx for C2, not like Pi
 - Processors, chips ...

# Preparation

![Pinouts](GPIO-Pinout-Diagram-all.png)

Connect the UART for main MMDMV communication, I2C for OLED type 3, and reset pins.

This HAT board needs soldering (shorting) of boot1 jumper to be able to program.

# SW Installation Steps

 - Flash the OS image and boot the board
 - Use direct HDMA monitor and keyoboard or 
 - Get remote SSH Wired or WiFi by a USB dongle:
```
nmcli d wifi
nmcli d wifi connect XXXX password YYYY
systemctl status ssh
```
 - Login as root:odroid 
 - Add armhf arch: https://unix.stackexchange.com/questions/625576/how-to-run-32-bit-armhf-binaries-on-64-bit-arm64-debian-os-on-raspberry-pi
```
dpkg --add-architecture armhf
dpkg --print-architecture
dpkg --print-foreign-architectures 
``` 
 - Update / add OS stuff:
```
apt-get -y update && apt-get -y upgrade && apt-get -y dist-upgrade
apt autoremove && apt autoclean

apt install -y libstdc++6:armhf
apt install -y linux-libc-dev:armhf
apt install -y libc6-dev:armhf
reboot
```
- Add needed packages:
```
apt install -y ntp avahi-daemon miniupnpc file zip git curl net-tools python2
ln /usr/bin/python2 /usr/bin/python
```
- Add and check PHP7.0: https://tecadmin.net/install-php-ubuntu-20-04/
```
apt install -y software-properties-common ca-certificates lsb-release apt-transport-https 
LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php  
apt update 
apt install -y php7.0-cli php7.0-common php7.0-fpm php7.0-json php7.0-mbstring php7.0-opcache php7.0-readline php7.0-zip

apt list --installed | grep -i php
systemctl status php7.0-fpm
```
- Get the essentials from my extraction:
```
cd /tmp
wget https://github.com/ve2opn/Pi-Star-Odroid-C2-C4/releases/download/1.0/pi-odro-c2-4.tgz
tar zxvf pi-odro-c2-4.tgz -C /
# For C4 only -  execute following
# mv /etc/pistar-release-C4 /etc/pistar-release
```
- Add 2 users: (password raspberry)
```
adduser pi-star
adduser mmdvm
usermod -aG sudo pi-star
usermod -aG sudo mmdvm

# edit the sudoers, add the lines:
pi-star ALL=(ALL) NOPASSWD: ALL
www-data ALL=(ALL) NOPASSWD: ALL
```
- Add nginx:
```
apt install -y nginx
# default - keep when asking
# use q = quit from choices within test if you loop there
# test
systemctl status nginx
# fix sites.enabled if needed (remove the default)
reboot
```
- Pi-star update - also test the pi-star user login
```
#login as pi-star
sudo su
pistar-update
#again:
pistar-update
#and again
pistar-update
pistar-upgrade
reboot
```
- Tweak add my script to run on startup
```
crontab -e
@reboot  /home/pi-star/z_my.sh
```
- Access the Web interface, change to your settings / restore conf.
- Services check:
```
systemctl daemon-reload
# systemctl enable xxxxx.service   (start, stop, status)
# remove if exist: (by miniupnpc install!)
rm /etc/systemd/system/multi-user.target.wants/pistar-upnp.service 
rm /etc/systemd/system/multi-user.target.wants/pistar-upnp.timer
# enable manually for test
systemctl enable dapnetgateway.service
systemctl enable dgidgateway.service
systemctl enable dmr2nxdn.service
systemctl enable dmr2ysf.service
systemctl enable dmrgateway.service
systemctl enable dstarrepeater.service
systemctl enable ircddbgateway.service
systemctl enable mmdvmhost.service
systemctl enable mobilegps.service
systemctl enable nxdngateway.service
systemctl enable nxdnparrot.service
systemctl enable p25gateway.service
systemctl enable p25parrot.service
systemctl enable pistar-ap.service
systemctl enable pistar-remote.service
systemctl enable pistar-upnp.service
systemctl enable pistar-watchdog.service
systemctl enable ysf2dmr.service
systemctl enable ysf2nxdn.service
systemctl enable ysf2p25.service
systemctl enable ysfgateway.service
systemctl enable ysfparrot.service
reboot
```
 - **For C2 only** : OLED on /dev/i2c-1, 3(SDA), 5(SCL) activate: (permanent)
 ```
modprobe aml_i2c
echo "aml_i2c" | sudo tee /etc/modules
```
- Add HAT FW upgrading capabilities:
```
apt install stm32flash
```
- Use this script for FW update
```
/home/pi-star/install_fw_hsdualhat.sh
```
- Enjoy
