# Pi-star for ODROID-C2 and C4

**This is a modification of the famous pi-star DMR hotspot project:**  

https://www.pistar.uk/  
https://github.com/AndyTaylorTweet  

The repos are modified and files from the images are used.


## Hat

<img src="https://raw.githubusercontent.com/phl0/MMDVM_HS_Dual_Hat/master/mmdvm_hs_dual_hat.png" width="200">

HW: https://github.com/phl0/MMDVM_HS_Dual_Hat/blob/master/hardware/r1.3/mmdvm_hs_dual_hat.pdf

FW: https://github.com/juribeparada/MMDVM_HS

## Boards

<img src="https://dn.odroid.com/homebackup/201602/ODROID-C2.png" width="200">

https://wiki.odroid.com/odroid-c2/odroid-c2

<img src="https://wiki.odroid.com/_media/odroid-c4/c4_k.jpg" width="200">

https://wiki.odroid.com/odroid-c4/odroid-c4

## OS

C2: [ubuntu-20.04-3.16-minimal-odroid-c2-20210201.img.xz](https://odroid.in/ubuntu_20.04lts/c2/ubuntu-20.04-3.16-minimal-odroid-c2-20210201.img.xz)

C4: [ubuntu-20.04-4.9-minimal-odroid-c4-hc4-20220228.img.xz](https://odroid.in/ubuntu_20.04lts/c4-hc4/ubuntu-20.04-4.9-minimal-odroid-c4-hc4-20220228.img.xz)

# Differences from Pi-star supported boards

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

 - Flash a fresh Ubuntu OS image and boot  
 - Use direct HDMI monitor and USB keyboard or   
 - Get remote SSH access by wired or WiFi USB dongle:  
```
nmcli d wifi
nmcli d wifi connect XXXX password YYYY
systemctl status ssh
```
 - Login as root:odroid 
 - Add armhf foreign architecture, see here:   
 https://unix.stackexchange.com/questions/625576/how-to-run-32-bit-armhf-binaries-on-64-bit-arm64-debian-os-on-raspberry-pi
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
```
 - Get the pi-star essentials from my extraction:
```
cd /tmp
wget https://github.com/ve2opn/Pi-Star-Odroid-C2-C4/releases/download/1.2/pi-odro-c2-4.tgz
# Just in case is not first time here
systemctl stop nginx
systemctl stop php7.0-fpm
tar zxvf pi-odro-c2-4.tgz -C /
```
 - Fix release strings, e.g. use editor: nano /etc/pistar-release
 
**Odroid C2:**
```
[Pi-Star]
Pi-Star_Build_Date = 08-Feb-2021
Version = 4.1.4
ircddbgateway =	20181222
dstarrepeater = 20181222
MMDVMHost = 20200615_Pi-Star_v4
kernel = 3.16
Hardware = Odroid-C2
```
**Odroid C4:** :
```
[Pi-Star]
Pi-Star_Build_Date = 08-Feb-2021
Version = 4.1.4
ircddbgateway =	20181222
dstarrepeater = 20181222
MMDVMHost = 20200615_Pi-Star_v4
kernel = 4.9
Hardware = Odroid-C4
```
 - Add packages:
```
apt install -y ntp avahi-daemon miniupnpc file zip git curl net-tools hostapd fake-hwclock iptables-persistent shellinabox parted rsync dosfstools python2 stm32flash
ln /usr/bin/python2 /usr/bin/python
``` 
 - Add PHP7.0: https://tecadmin.net/install-php-ubuntu-20-04/
```
apt install -y software-properties-common ca-certificates lsb-release apt-transport-https 
LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php  
apt update 
apt install -y php7.0-cli php7.0-common php7.0-fpm php7.0-json php7.0-mbstring php7.0-opcache php7.0-readline php7.0-zip
apt list --installed | grep -i php
systemctl status php7.0-fpm
```
 - Add 2 users: (password raspberry):  
```
adduser pi-star
adduser mmdvm
usermod -aG sudo pi-star
usermod -aG sudo mmdvm

# check the sudoers last lines should be like  pi-star ALL=(ALL) NOPASSWD: ALL www-data ALL=(ALL) NOPASSWD: ALL
visudo
# vi quitting is by Ctrl+
```
 - Add nginx:
```
apt install -y nginx
# default - keep when asking
# use q = quit from choices within test if you loop there
#
rm /etc/nginx/sites-enabled/default
systemctl start nginx
systemctl status nginx
```
 - Pi-star update, upgrade and add a script to run on startup:  
```
#pistar-update -f triggers OS update as well
pistar-update
#again:
pistar-update
pistar-upgrade
#
crontab -e
# add a line at the end: 
@reboot  /home/pi-star/z_my.sh
```
 - Access the Web interface, activate the hat, change to your settings / restore conf. if you have one.
 - Check the modem and enable services:  
```
pistar-findmodem
systemctl daemon-reload
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
systemctl enable timeserver.service
systemctl enable ysf2dmr.service
systemctl enable ysf2nxdn.service
systemctl enable ysf2p25.service
systemctl enable ysfgateway.service
systemctl enable ysfparrot.service
reboot
```
 - **Odroid C2 only** : OLED on /dev/i2c-1, 3(SDA), 5(SCL) activate i2C-1: (permanent)
 ```
modprobe aml_i2c
echo "aml_i2c" | sudo tee /etc/modules
```
- Use these scripts for HAT Firmware update (may need editing for another HAT or FW rev.):
```
/usr/local/bin/install_fw_hsdualhat_local_152.sh
# or
/home/pi-star/install_fw_hsdualhat.sh
``` 
 - **Additional notes**  
a) Odroid C4 has issues with I2C so the OLED type 3 may not work.  
b) To ensure better startup, you can add a reset mmdvm hat line to **/home/pi-star/z_my.sh**
```
pistar-mmdvmhshatreset
# you can add pistar-update here, too
# comment all lines like /usr/local/sbin/aprsgateway.service start
```
c) **pistar-clone-c2** or **pistar-clone-c4** commands can be used to create a SD card image that fits to 8GB+ size  
You can try to fit to 4GB with command flag -4  

*Warning - it may destroy your working partition in case of errors!*  
*Next verifications are optional, e.g. if something is wrong*  
**Verify UUID:** Mount the newly cloned partition and find by **blkid** command the UUID="xxxx....,   
Example, /dev/sdc2: UUID="96d8a621-b8f2-45b9-8f95-35bdbb83afc7" TYPE=...
```
blkid 
```
Check the line in the new **boot.ini** on cloned SD' first partition to match the cloned root partition UUID
```
setenv bootargs "root=UUID=96d8a621-b8f2-45b9-8f95-35bdbb83afc7 ...
```
**Also:** check for matching UUID in /etc/fstab  

If you have a large SD card, you can expand the root to take unused space by:  
```
pistar-expand
```

d) You can do following to recreate an updated essentials tgz file to /tmp:    
Use the file-list from this repo:
```
wget --no-check-certificate -O /tmp/file-list https://raw.githubusercontent.com/ve2opn/Pi-Star-Odroid-C2-C4/main/file-list
tar zcvf /tmp/pi-odro-c2-4.tgz -T /tmp/file-list
```
e) **firewall: iptables-persistent** - firewall stuff
```
pistar-firewall
netfilter-persistent save
```
f) Enable the NextionDriver (for MMDVMHost) if you use such screen to enhance see:   
https://github.com/PD0DIB/NextionDriver#readme  
https://github.com/WA6HXG  
http://www.hs9awo.com/nextion1/

You can add a line to **/home/pi-star/z_my.sh**  
```
/usr/local/sbin/nextiondriver.service start
```


## Useful links to setup your DMR Gateway 

https://github.com/g4klx/DMRGateway/wiki/Rewrite-Rules  
https://freestar.network/dmrplus-options-explained/  
https://freestar.network/tools/dmrplus-options-generator.php  
https://www.freedmr.uk/index.php/static-talk-groups-pi-star/

## Next
Similar boards to explore, they should work same way , example AML-S905X-CC, see 

[Here](https://www.aliexpress.com/item/1005005163398168.html)

[Here](https://www.loverpi.com/products/libre-computer-aml-s905x-cc-le-potato-with-heatsink-and-wifi-4?variant=39845153701946)


 - Enjoy
