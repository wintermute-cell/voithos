# voithos
A writeup on how to setup a system like voithos, including installing archlinux on a Raspberry Pi 3B

## Required supplies
- SD-card
- Raspberry Pi 3B
- Raspberry Pi 3 compatible Power Supply (Careful, Pi3 draws more power than 2!)

## Partitioning the SD-card
Run ```lsblk``` to find the SD-cards name, I'll call it sdX from now on.

Then run ```fdisk /dev/sdX```:
Using fdisk delete all old partitions and create a new one:
- ```o```to delete all existing partitions from the drive.
- ```p``` to list partitions. Should return an empty list is the clearing was successful.
- ```n```, to make a new partition, then ```p``` for type = primary, ```1``` as the partition number, and then press <ENTER> to accept the default first sector, then type ```+200M``` (200MBs) for the last sector.
- ```t```, to set the partition type. Enter ```c``` to set the first partition to type W95 FAT32 (LBA).
- ```n```, to create another new partition, then enter ```p``` for primary, ```2``` as the partition number, and then press <ENTER>  to accept both default first, and last sector.
- Using ```w``` write the created partition table to the disk; this also simultaneously exits fdisk.

Create and mount the FAT filesystem:
- ```mkfs.vfat /dev/sdX1```
- ```mkdir boot```
- ```mount /dev/sdX1 boot```

Create and mount the ext4 filesystem:
- ```mkfs.ext4 /dev/sdX2```
- ```mkdir root```
- ```mount /dev/sdX2 root```

### Sources:
    - https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3
