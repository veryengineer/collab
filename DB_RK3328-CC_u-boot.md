########################FDISK for Ubuntu Jammy/Lunar arm64

    Run fdisk as root: sudo fdisk /dev/sdb
    Type g to set GPT Table
    Type p to list the current partitions.
    Note the starting sector of each partition you want to recreate (/dev/sdb14 and /dev/sdb15).
    Type d to delete each partition, starting with /dev/sdb15, then /dev/sdb14, and finally /dev/sdb1.
    Type n to create a new partition.
    Type p to create a primary partition.
    Enter 1 as the partition number.
    Enter 235457 as the starting sector (the same as the previous /dev/sdb1 partition).
    For the ending sector, choose default.
    Type n again to create the next partition.
    Type p to create a primary partition.
    Enter 14 as the partition number.
    Enter 227265 as the starting sector (the same as the previous /dev/sdb14 partition).
    Enter 235456 as the ending sector (the same as the previous /dev/sdb14 partition).
    Type n again to create the next partition.
    Type p to create a primary partition.
    Enter 15 as the partition number.
    Enter 24512 as the starting sector (the same as the previous /dev/sdb15 partition).
    Enter 227264 as the ending sector (the same as the previous /dev/sdb15 partition).
    Type w to write the changes to disk.

After completing these steps, you should have approximately 13MB of empty space at the beginning of the image for a bootloader.


##################Erase with 000000000000000000000

sudo dd if=/dev/zero of=/dev/sdb1 bs=1M


dd: error writing '/dev/sdb1': No space left on device
3667+0 records in
3666+0 records out
3844619264 bytes (3.8 GB, 3.6 GiB) copied, 614.812 s, 6.3 MB/s



sudo dd if=/dev/zero of=/dev/sdb14 bs=1M


dd: error writing '/dev/sdb14': No space left on device
5+0 records in
4+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 2.98357 s, 1.4 MB/s



sudo dd if=/dev/zero of=/dev/sdb15 bs=1M


dd: error writing '/dev/sdb15': No space left on device
100+0 records in
99+0 records out
103809536 bytes (104 MB, 99 MiB) copied, 17.8615 s, 5.8 MB/s


erase boot loader:


sudo dd if=/dev/zero of=/dev/sdb bs=512 seek=64 count=24447

24447+0 records in
24447+0 records out
12516864 bytes (13 MB, 12 MiB) copied, 7.51947 s, 1.7 MB/s



###################################### Cloning

Download ubuntu lunar preinstalled server: 

https://mirrors.cloud.tencent.com/ubuntu-cdimage/ubuntu-server/daily-preinstalled/current/lunar-preinstalled-server-arm64.img.xz

Extract and mount image with:


sudo losetup -fP --show lunar-preinstalled-server-arm64.img


Clone each partition to each corresponding partition on the removable media.


sudo dd if=/dev/loop0p1 of=/dev/sb1 bs=1M status=progress

sudo dd if=/dev/loop0p14 of=/dev/sb14 bs=1M status=progress

sudo dd if=/dev/loop0p15 of=/dev/sb15 bs=1M status=progress


At this point I create an image AFTER you manage the bootflags to match the original image (boot, esp, legacy boot); this way the image has no bootloader.


######################### Flash compiled boot loader

sudo dd if=u-boot-rockchip.bin of=/dev/sdb seek=64





