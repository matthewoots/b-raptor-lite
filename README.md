# B-RAPTOR-LITE
Development repository for sub 250g quadcopter with
- `Onboard computer` (**Bananapi-M2-Zero**)
- `VIO` (using **Basalt** [here](https://github.com/VladyslavUsenko/basalt-mirror))
- `Basalt-wrapper` (using [repo](https://github.com/berndpfrommer/basalt_ros))
- `PX4` (connection using **mavros** and PX4 [repo](https://github.com/PX4/PX4-Autopilot))

## Bananapi-M2-Zero Setup
Preferably use a 64GB MicroSD card (**todo**)

### 1.1 Clone Image (For existing image)
```bash
# To clone pre-existing bpi image
# Should show 1. Boot partition (/dev/mmcblk0p1) 2. Root file system partition (/dev/mmcblk0p2)
lsblk -p 

# $SD_CARD_DEVICE_NAME = /dev/sdX shown on lsblk -p output
# $IMAGE_FILE_NAME = ~/Downloads/sample.img
sudo dd bs=4M if=[$SD_CARD_DEVICE_NAME] of=[$IMAGE_FILE_NAME] conv=fsync
sudo chown $USER: [$IMAGE_FILE_NAME]

# Installation of PiShrink
cd ~
wget https://raw.githubusercontent.com/Drewsif/PiShrink/master/pishrink.sh
chmod +x pishrink.sh
sudo mv pishrink.sh /usr/local/sbin/pishrink
sudo pishrink [$IMAGE_FILE_NAME]

cd ~/Downloads/
tar -czf [$IMAGE_NAME].tar.gz [$IMAGE_NAME].img
```

Use **BalenaEtcher** to format and write over a new microSD card

### 1.2 Starting from Scratch
1. Thanks to Avafinger, download from [Avafinger's repository](https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/tag/v3.10) the release for **Ubuntu 20.04**
```bash 
# Get release
mkdir ~/bpi_img
cd ~/bpi_img
wget https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/download/v3.10/bootloader_m2z_v2.bin
wget https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/download/v3.10/boot_focal.tar.gz
wget https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/download/v3.10/flash_sdcard_mz2_focal.sh
wget https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/download/v3.10/format_sd_mainline.sh
wget https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/download/v3.10/rootfs_focal.tar.gz

# Make executables
sudo chmod +x format_sd_mainline.sh
sudo chmod +x flash_sdcard_mz2_focal.sh
lsblk -p 

# Flashing to SD card
# Make sure lsblk -p gives the right output like /dev/sdX
./format_sd_mainline.sh /dev/sdX
./flash_sdcard_mz2_focal.sh /dev/sdX
```

2. Change 4 things **before you boot**

- *Network* : Edit the `/etc/netplan/01-netcfg.yaml` and change your **ssid** (`FIBER-8784`) and **password**
- *Sources* : Edit the `/etc/apt/sources.list`, take out the original **x64** and add in the packages below for **armhf**
- *Autologin* : Edit the `rootfs/usr/lib/systemd/system/getty@.service`, and find the `ExecStart=` line and replace it with `ExecStart=-/sbin/agetty -o ubuntu --noclear %I $TERM`
- *Swap* : Follow the instruction below to increase swap preferably to **2GB**

```
# /etc/apt/sources.list

#deb http://archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse
#deb-src http://archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse

#deb http://archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse
#deb-src http://archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse

#deb http://archive.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse
#deb-src http://archive.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse

#deb http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
#deb-src http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse

deb http://archive.canonical.com/ubuntu focal partner
deb-src http://archive.canonical.com/ubuntu focal partner

deb [arch=amd64,i386] http://us.archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse
deb [arch=amd64,i386] http://us.archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse
deb [arch=amd64,i386] http://us.archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
deb [arch=amd64,i386] http://security.ubuntu.com/ubuntu focal-security main restricted universe multiverse

deb [arch=arm64,armhf,ppc64el,s390x] http://ports.ubuntu.com/ubuntu-ports/ focal main restricted universe multiverse
deb [arch=arm64,armhf,ppc64el,s390x] http://ports.ubuntu.com/ubuntu-ports/ focal-updates main restricted universe multiverse
deb [arch=arm64,armhf,ppc64el,s390x] http://ports.ubuntu.com/ubuntu-ports/ focal-backports main restricted universe multiverse
deb [arch=arm64,armhf,ppc64el,s390x] http://ports.ubuntu.com/ubuntu-ports/ focal-security main restricted universe multiverse
```

```
# Swap Size
# Check swap size
sudo swapon --show
free -h
df -h

# Start Allocating
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
ls -lh /swapfile

sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon --show
free -h

# Making it permanent
sudo cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Improve swappiness and cache pressure
sudo nano /etc/sysctl.conf
# Add the 2 lines below
# Swappiness = This is generally very costly to look up and very frequently requested, so it’s an excellent thing for your system to cache
# 
# vm.swappiness=10
# vm.vfs_cache_pressure=50
```

