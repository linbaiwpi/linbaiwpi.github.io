## How to format SD card and run ZCU104 with pre-built petalinux
Date: 2019-03-27

This tutorial is based on Xilinx Wiki [How to format SD card for SD boot
](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842385/How+to+format+SD+card+for+SD+boot) and [Zynq 2018.3 Release](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/57639129/Zynq+2018.3+Release)

1. Unmount the sd card by command
```sh
umount /dev/sdb1
```
2. Follow the tutorial mentioned above to partition the sd card and write
3. Format partitions
4. Download the pre-built linux "2018.3-zcu104-release.tar.xz"
5. Copy the content into 'boot' drive
6. Insert sd card and boot the board

# Possible Issues
**Q** Re-reading the partition table failed.: Device or resource busy
![](/image/zcu104_tut/tut1_1.jpg)
**A** the device is mounted. run 
```sh
umount /dev/<your_sd_card> 
```
and try again

