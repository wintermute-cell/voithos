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

Now, to make things easier, temporarily set your keyboard layout with ```loadkeys *LAYOUT*```. You can list available layouts with ```ls /usr/share/kbd/keymaps/**/*.map.gz```

If you want use SSH, wait for it to finish booting, then run enable ssh with the following:

```systemctl enable sshd.service```  
```systemctl start sshd.service```

and then SSH into the device, using the IP adress given by your router.

Log in using the default user ```alarm``` with the default password ```alarm```.

Root password is ```root``` by default. (Used for "alarmpi login:" and the following "password:")

Initialize the pacman keyring ```pacman-key --init``` and populate the Arch Linux arm package signing keys: ```pacman-key --populate archlinuxarm```

Done! You've now got a pretty basic, but running archlinux installation! At this point, you can choose to follow this guide further, to learn how I set up my arch install, or go your own way from here on!

## Basic setup
### Setting the time
First, in order to set the timezone, run ```ln -sf /usr/share/zoneinfo/REGION/CITY /etc/localtime```. use ```ls``` in these directories to search for you zone.
Run ```systemctl start systemd-timesyncd.service``` and ```systemctl enable systemd-timesyncd.service```, and then ```timedatectl set-ntp true``` to enable time & date sync over the internet.

### Full system upgrade
Run ```pacman -Syu``` to perform a full system upgrade.

### (Optional) Installing a text editor
Arch comes with ```vi``` preinstalled. If you'd like to proceed with ```vim```, ```nvim```, ```nano```, etc...,  then install the desired package now.

### Setting up wifi
After correctly setting the time, install ```networkmanager```, and start/enable ```NetworkManager.service``` using ```systemctl```.

Then run ```nmcli device wifi list``` to list available wireless access points, followed by ```nmcli device wifi connect *SSID* password *PASSWORD*``` to connect to the network. Now finally run ```nmcli connection up *SSID*```. 

### Setting up sudo
Using pacman, install ```sudo```. Then, run ```visudo``` to edit the sudoers file. To enable full sudo permissions for all users in the ```wheel``` group, uncomment either  
```# %wheel ALL=(ALL) ALL```  
or  
```# %wheel ALL=(ALL) NOPASSWD: ALL```  
depending on wether you want a password promt or not.

### Creating a new user
To create a new user with administrative permissions, run ```useradd -m -g wheel *USERNAME*```. ```-m``` creates a home directory for the new user, and the wheel group is required for certain permissions (see *"Setting up sudo"*).

### A note on the swap partition
An SD-card has a very limited lifetime, especially when written to frequently. Using swap (particularly for hibernation) has a negative effect on SD-card lifetime. My recommendation would be to restrain from using swap, and to just be a little careful managing memory consumption. Should you choose to use swap anyway, see "Partition the disks" at https://wiki.archlinux.org/title/installation_guide.

### A note on hwclock
The rpi3 doesnt have a hardware clock, don't bother with hwclock. YOu can get one with external components though.

### Localization
Edit ```/etc/locale.gen```, uncommenting your desired locale (remember the name of the locale, you'll need it in the next step).  
Then run ```locale-gen``` to generate the locale.

Now edit ```/etc/locale.conf```, setting  
```LANG=LOCALE_NAME``` (for example ```de_DE.UTF-8```.)

To permanently set your keyboard layout, create/edit ```/etc/vconsole.conf``` adding the line ```KEYMAP=*LAYOUT*``` where *LAYOUT* is your desired layout. You can list available layouts with ```ls /usr/share/kbd/keymaps/**/*.map.gz```.

### Network
To setup the machines hostname, edit ```/etc/hostname```, and simply enter your desired hostname as the first and only word of that file.

Then, edit ```/etc/hosts```, adding
> 127.0.0.1	localhost
> ::1		localhost
> 127.0.1.1	HOSTNAME.localdomain	HOSTNAME  
where *HOSTNAME* is the hostname you have set in the previous step.

## Unclear
### Time issue
*Starting ```systemctl start systemd-timesyncd.service``` just worked at some point?? After that ```timedatectl set-ntp true``` to enable time sync over internet*. Could have had something to do with 
### pacman key issue
*Seemingly had to do ```pacman-key --init``` and ```pacman-key --populate archlinuxarm``` again after setting time?*


## Sources:
- https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3
- https://www.raspberrypi.org/forums/viewtopic.php?t=90337
- https://wiki.archlinux.org/title/systemd
- https://wiki.archlinux.org/title/NetworkManager
- https://askubuntu.com/questions/795226/how-to-list-all-enabled-services-from-systemctl
- https://wiki.archlinux.org/title/installation_guide
- 
