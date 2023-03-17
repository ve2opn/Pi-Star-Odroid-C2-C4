# Pi-star for ODROID-C2 and C4

Inspired from / credits: https://www.pistar.uk/

## Hat

HW: https://github.com/phl0/MMDVM_HS_Dual_Hat/blob/master/hardware/r1.3/mmdvm_hs_dual_hat.pdf
FW: https://github.com/juribeparada/MMDVM_HS

## Boards

https://wiki.odroid.com/odroid-c2/odroid-c2
https://wiki.odroid.com/odroid-c4/odroid-c4

## OS

C2: [ubuntu-20.04-3.16-minimal-odroid-c2-20210201.img.xz](https://odroid.in/ubuntu_20.04lts/c2/ubuntu-20.04-3.16-minimal-odroid-c2-20210201.img.xz)
C4: [ubuntu-22.04-4.9-minimal-odroid-c4-hc4-20220705.img.xz](https://odroid.in/ubuntu_22.04lts/C4_HC4/ubuntu-22.04-4.9-minimal-odroid-c4-hc4-20220705.img.xz)

# Differences

 - [Pi-star](https://www.pistar.uk/downloads/) provide **armhf** images for Pi and ODROID-XU4. Unfortunately, these don't run directly on C2 and C4 boards, which use **aarch64 (arm64)** OS.
 - Ubuntu images for C2/C4 mount a partition to **/media/boot** not to **/boot** so many pi-star scripts need a modification
 - 40-pin connectors on C2 and C4 look similar to Pi but they are different - especially the pins for hat reset / boot.
 - GPIO# are different for each board i.e. GPIO2xx for C2, not like Pi

# Challenges

# Steps
