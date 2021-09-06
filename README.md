# voithos
A writeup on how to setup a system like voithos, including installing archlinux on a Raspberry Pi 3B

## Required supplies
- SD-card
- Raspberry Pi 3B
- Raspberry Pi 3 compatible Power Supply (Careful, Pi3 draws more power than 2!)
- Ethernet cable (and a free port on your rounter ofc!)

## Partitioning the SD-card
Run ```lsblk``` to find the SD-cards name, I'll call it sdX from now on.

Then run ```fdisk /dev/sdX```:
Using fdisk delete all old partitions and create a new one:
- ```o```to delete all existing partitions from the drive.
- ```p``` to list partitions. Should return an empty list is the clearing was successful.
- ```n```, to make a new partition, then ```p``` for type = primary, ```1``` as the partition number, and then press ENTER to accept the default first sector, then type ```+200M``` (200MBs) for the last sector.
- ```t```, to set the partition type. Enter ```c``` to set the first partition to type W95 FAT32 (LBA).
- ```n```, to create another new partition, then enter ```p``` for primary, ```2``` as the partition number, and then press ENTER to accept both default first, and last sector.
- Using ```w``` write the created partition table to the disk; this also simultaneously exits fdisk.

Create and mount the FAT filesystem:
- ```mkfs.vfat /dev/sdX1```
- ```mkdir boot```
- ```mount /dev/sdX1 boot```

Create and mount the ext4 filesystem:
- ```mkfs.ext4 /dev/sdX2```
- ```mkdir root```
- ```mount /dev/sdX2 root```

## Installing the filesystem
Visit https://archlinuxarm.org/about/downloads to find the correct version for your model.

At the moment, http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz is the correct one for my device. (Ignore the "rpi-2" in the link, it's fine)

Download the package using ```wget http://os.archlinuxarm.org/os/[YOUR_CHOSEN_VERSION_HERE].tar.gz```.

Extract the package into the root dir using ```tar -xf ArchLinuxARM-rpi-2-latest.tar.gz -C root```

Now ```sync``` to move the files to the SD-card.

Move the boot files to the corresponding directory using ```sudo mv boot/* boot```

Unmount the two partitions using ```umount root boot```

## Setting up the device
Move the SD-card to the Pi and connect via ethernet cable. 

Now connect the HDMI cable.

Plug in the power cable! (And watch it boot)

If you want use SSH, wait for it to finish booting, then run enable ssh with the following:

```systemctl enable sshd.service```

```systemctl start sshd.service```

and then SSH into the device, using the IP adress given by your router.

Log in using the default user ```alarm``` with the default password ```alarm```.

Root password is ```root``` by default. (Used for "alarmpi login:" and the following "password:")

Initialize the pacman keyring ```pacman-key --init``` and populate the Arch Linux arm package signing keys: ```pacman-key --populate archlinuxarm```

Done! You've now got a pretty basic, but running archlinux installation! At this point, you can choose to follow this guide further, to learn how I set up my arch install, or go your own way from here on!

## Under Construction
### Time issue
*Starting ```systemctl start systemd-timesyncd.service``` just worked at some point?? After that ```timedatectl set-ntp true``` to enable time sync over internet*.


## Sources:
- https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3
- https://www.raspberrypi.org/forums/viewtopic.php?t=90337
