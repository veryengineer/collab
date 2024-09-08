# u-boot compile branch 2023.04 for roc-cc-rk3328


### Dependencies

```
sudo apt-get install gcc gcc-aarch64-linux-gnu git bc bison build-essential coccinelle device-tree-compiler dfu-util efitools flex gdisk graphviz imagemagick liblz4-tool libgnutls28-dev libguestfs-tools libncurses-dev libpython3-dev libsdl2-dev libssl-dev lz4 lzma lzma-alone openssl pkg-config python3 python3-asteval python3-coverage python3-filelock python3-pkg-resources python3-pycryptodome python3-pyelftools python3-pytest python3-pytest-xdist python3-sphinxcontrib.apidoc python3-sphinx-rtd-theme python3-subunit python3-testtools python3-virtualenv swig uuid-dev
```

### Update

```
sudo apt update
sudo apt dist-upgrade
```

## Clone Arm Trusted Firmware Repo and compile

```
git clone --depth 1 https://github.com/ARM-software/arm-trusted-firmware.git
cd arm-trusted-firmware
make realclean

make CROSS_COMPILE=aarch64-linux-gnu- PLAT=rk3328
```

### Exit directory

```
cd ..
```

## Clone Libretech U-Boot branch

```
git clone -b "v2023.04/roc-rk3328-cc" https://github.com/libre-computer-project/libretech-u-boot.git
cd libretech-u-boot
```

### Export Arm Trusted

```
export BL31=../arm-trusted-firmware/build/rk3328/release/bl31/bl31.elf
```

### Define the configuration

```
make roc-cc-rk3328_defconfig
```

### Compile bootloader files

```
make CROSS_COMPILE=aarch64-linux-gnu-
```


## Prepare SD Card

Insert SD-Card into your Cardreader or USB-Cardreader Dongle and plug it in a free USB Port on your Machine.


### Run "lsblk" to asure, it is mounted as mass storage device "sdb":

```
sudo lsblk
```

In case the SD-Card shows up as a different device, (eg.: sdc) adapt the codelines accordingly before running them!


### Partitioning the SD-Card

To manually add the correct Partitiontables for booting the Image run FDISK for Ubuntu Jammy/Lunar (/arm64)


Run fdisk as root:

```
sudo fdisk /dev/sdb`
```

Follow these Steps:

```
Type g to set GPT Table
Type p to list the current partitions.
Note the starting sector of each partition you want to recreate (/dev/sdb14 and /dev/sdb15).
Type n to create a new partition.
Enter 1 as the partition number.
Enter 235457 as the starting sector (the same as the previous /dev/sdb1 partition).
For the ending sector, choose default.
Type n again to create the next partition.
Enter 14 as the partition number.
Enter 227265 as the starting sector (the same as the previous /dev/sdb14 partition).
Enter 235456 as the ending sector (the same as the previous /dev/sdb14 partition).
Type n again to create the next partition.
Enter 15 as the partition number.
Enter 24512 as the starting sector (the same as the previous /dev/sdb15 partition).
Enter 227264 as the ending sector (the same as the previous /dev/sdb15 partition).
Type w to write the changes to disk.
```

After completing these steps, you should have approximately 13MB of empty space at the beginning of the image for a bootloader.


### Prime/Erase the SD-Card with 000000000000000000000

```console
$ sudo dd if=/dev/zero of=/dev/sdb1 bs=1M

#dd: error writing '/dev/sdb1': No space left on device
#3667+0 records in
#3666+0 records out
#3844619264 bytes (3.8 GB, 3.6 GiB) copied, 614.812 s, 6.3 MB/s

$ sudo dd if=/dev/zero of=/dev/sdb14 bs=1M

#dd: error writing '/dev/sdb14': No space left on device
#5+0 records in
#4+0 records out
#4194304 bytes (4.2 MB, 4.0 MiB) copied, 2.98357 s, 1.4 MB/s

$ sudo dd if=/dev/zero of=/dev/sdb15 bs=1M

#dd: error writing '/dev/sdb15': No space left on device
#100+0 records in
#99+0 records out
#103809536 bytes (104 MB, 99 MiB) copied, 17.8615 s, 5.8 MB/s
```

### Erase the bootloader

```console
$ sudo dd if=/dev/zero of=/dev/sdb bs=512 seek=64 count=24447

#24447+0 records in
#24447+0 records out
#12516864 bytes (13 MB, 12 MiB) copied, 7.51947 s, 1.7 MB/s

```

## Cloning Oracular server


### Download Ubuntu Oracular preinstalled Server:

```
sudo wget https://mirrors.cloud.tencent.com/ubuntu-cdimage/ubuntu-server/daily-preinstalled/current/oracular-preinstalled-server-arm64.img.xz
```


### Extract and mount the .img

```
sudo xz -d oracular-preinstalled-server-arm64.img.xz

sudo losetup -fP --show oracular-preinstalled-server-arm64.img
```

### Clone each partition to each corresponding partition on the removable media.


Run lsblk and check the mountpoint of the p1,p14 & p15 loops.

```
sudo lsblk
```


By default those should be on loop0.


### In case of a different loop, please substitute accordingly before running the command below.


```
sudo dd if=/dev/loop0p1 of=/dev/sdb1 bs=1M status=progress
sudo dd if=/dev/loop0p14 of=/dev/sdb14 bs=1M status=progress
sudo dd if=/dev/loop0p15 of=/dev/sdb15 bs=1M status=progress
```

At this point I create an image AFTER you manage the bootflags to match the original image (boot, esp, legacy boot); this way the image has no #bootloader.


## Flash compiled boot loader


```
cd libretech-u-boot
sudo dd if=u-boot-rockchip.bin of=/dev/sdb seek=64
```

# Cleanup

For Cleaning up the previously downloaded .img and Github Directories, run the following commands:


```
cd ..
sudo rm -rf ./oracular-preinstalled-server-arm64.img.xz ./libretech-u-boot/ ./arm-trusted-firmware/
```
