# voithos
A writeup on how to setup a system like voithos, including installing archlinux on a Raspberry Pi 3B

## Required supplies
- SD-card
- Raspberry Pi 3B
- Raspberry Pi 3 compatible Power Supply (Careful, Pi3 draws more power than 2!)

## Partitioning the SD-card
Run ```lsblk``` to find the SD-cards name, I'll call it sdX from now on.
Run ```fdisk /dev/sdX```:
Using fdisk delete all old partitions and create a new one:

    Type o. This will clear out any partitions on the drive.
    Type p to list partitions. There should be no partitions left.
    Type n, then p for primary, 1 for the first partition on the drive, press ENTER to accept the default first sector, then type +200M for the last sector.
    Type t, then c to set the first partition to type W95 FAT32 (LBA).
    Type n, then p for primary, 2 for the second partition on the drive, and then press ENTER twice to accept the default first and last sector.
    Write the partition table and exit by typing w.

Create and mount the FAT filesystem:

  > mkfs.vfat /dev/sdX1
  > mkdir boot
  > mount /dev/sdX1 boot

Create and mount the ext4 filesystem:

mkfs.ext4 /dev/sdX2
mkdir root
mount /dev/sdX2 root

