Overview


This document provides instructions for building shell code in a deterministic way. A deterministic build ensures that the resulting binary is 
identical every time it is built, regardless of the environment or the build system used.


Requirements


The following tools and dependencies are required to verify and write the shell code in a deterministic way:

    Bash shell
    Git
    sha256sum
    Zip
    Tar


Building the Image:


Follow these steps to build the bootable disk image in a deterministic way:



Download and confirm the checksum matches AFTER EXTRACTING the u-boot-lunar.bin.zip by opening a shell in the same directory of the downloaded image file 
and by executing the following command:

[u-boot-lunar.bin.zip](https://github.com/veryengineer/collab/files/11195583/u-boot-lunar.bin.zip)


``
sha256sum u-boot-lunar.bin
``

1fbc136350ae81c5dd1dc5bda35fc7d04c6a41cbcc1952fd36fe38794f440e34  u-boot-lunar.bin



[Download the lunar-preinstalled-server-arm64.img](https://mirrors.cloud.tencent.com/ubuntu-cdimage/ubuntu-server/daily-preinstalled/current/lunar-preinstalled-server-arm64.img.xz)


Confirm the checksum matches AFTER EXTRACTING the lunar-preinstalled-server-arm64.img by opening a shell in the same directory of the downloaded file and by
executing the following command:


``
sha256sum lunar-preinstalled-server-arm64.img
``

68d6f785c263bbfa472cee5f221e1c97a616a5cfedbc6f78a7e7843a60980cd6  lunar-preinstalled-server-arm64.img




Once checksums have been confirmed, open a shell in the same directory with both u-boot-lunar.bin and the ubuntu-lunar-preinstall-server-arm64.img

Write the u-boot-lunar.bin bootloader to the image file with the following:

``
dd if=u-boot-lunar.bin of=lunar-preinstalled-server-arm64.img bs=512 seek=64 count=1984 conv=notrunc
``

Write the image to a microSD via USB adapter with the following:

``
dd if=lunar-preinstalled-server-arm64.img of=/dev/sdX bs=512 seek=64 count=1984 conv=notrunc
``

Conclusion

By following these instructions, you can build the image in a deterministic way, ensuring that the resulting binary is identical every time it is built. 
This can help improve reproducibility and reliability of the build process.



