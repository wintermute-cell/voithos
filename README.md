# voithos
installing archlinux on a Raspberry Pi 3B

![alt text](https://raw.githubusercontent.com/wintermute-cell/voithos/main/screensh.png)

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
First, in order to set the timezone, run ```ln -sf /usr/share/zoneinfo/REGION/CITY /etc/localtime```.  
Use ```ls``` in these directories to search for you zone.  
Run ```systemctl start systemd-timesyncd.service``` and ```systemctl enable systemd-timesyncd.service```,  
and then ```timedatectl set-ntp true``` to enable time & date sync over the internet.  
Edit `/etc/systemd/timesyncd.conf`:  

```
[Time]
NTP=YOUR.SERVERS.1 YOUR.SERVERS.2 (for example 0.de.pool.ntp.org 1.de.pool.ntp.org)
FallbackNTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
```
It is likely that `systemd-networkd` starts after `systemd-timesyncd`, causing the time to not syncronize properly. To fix this, use `systemdctl edit systemd-networkd`.
In the openend file, read the comments. In the instructed are, add the following:
```
[Unit]
After=systemd-timesyncd.service
``` 

### Full system upgrade
Run ```pacman -Syu``` to perform a full system upgrade.

### (Optional) Installing a text editor
Arch comes with ```vi``` preinstalled. If you'd like to proceed with ```vim```, ```nvim```, ```nano```, etc...,  then install the desired package now.

### Setting up wifi
After correctly setting the time, install ```networkmanager```, and start/enable ```NetworkManager.service``` using ```systemctl```.

Then run ```nmcli device wifi list``` to list available wireless access points, followed by ```nmcli device wifi connect *SSID* password *PASSWORD*``` to connect to the network. Now finally run ```nmcli connection up *SSID*```. 

### A note on the swap partition
An SD-card has a very limited lifetime, especially when written to frequently. Using swap (particularly for hibernation) has a negative effect on SD-card lifetime. My recommendation would be to restrain from using swap, and to just be a little careful managing memory consumption. Should you choose to use swap anyway, see "Partition the disks" at https://wiki.archlinux.org/title/installation_guide.

### A note on hwclock
The rpi3 doesnt have a hardware clock, don't bother with hwclock. You can get one with external components though.

### Localization
Edit ```/etc/locale.gen```, uncommenting your desired locale (remember the name of the locale, you'll need it in the next step).  
Then run ```locale-gen``` to generate the locale.

To set the system language edit ```/etc/locale.conf```, setting  
```LANG=LOCALE_NAME``` (for example ```en_US.UTF-8```.)

To permanently set your keyboard layout, create/edit ```/etc/vconsole.conf``` adding the line ```KEYMAP=*LAYOUT*``` where *LAYOUT* is your desired layout. You can list available layouts with ```ls /usr/share/kbd/keymaps/**/*.map.gz```.

### Network
To setup the machines hostname, edit ```/etc/hostname```, and simply enter your desired hostname as the first and only word of that file.

Then, edit ```/etc/hosts```, adding

    127.0.0.1	localhost
    ::1		localhost
    127.0.1.1	HOSTNAME.localdomain	HOSTNAME

where *HOSTNAME* is the hostname you have set in the previous step. If the device has a permanent IP adress, use that one instead of `127.0.1.1`.

### Setting the root password
Type `passwd` to change the password of the root user if desired.

### Setting up sudo
Using pacman, install ```sudo```. Then, run ```visudo``` to edit the sudoers file. To enable full sudo permissions for all users in the ```wheel``` group, uncomment either  
```# %wheel ALL=(ALL) ALL```  
or  
```# %wheel ALL=(ALL) NOPASSWD: ALL```  
depending on wether you want a password promt or not.

### Creating a new user
To create a new user with administrative permissions, run ```useradd -m -g wheel *USERNAME*```. ```-m``` creates a home directory for the new user, and the wheel group is required for certain permissions (see *"Setting up sudo"*).

To switch to that new user, use `su USERNAME`. To get back to the root user, use `su root`.

### Disabling overscan (getting rid of screen borders)
Edit `/boot/config.txt`, appending this line:  
```
disable_overscan=1
```

This will get rid of the screen borders after a reboot.

### Reducing access to the SD-card
As root, do the following:  
To instruct systemd to store it's journals in RAM, first do `mkdir /etc/systemd/journald.conf.d/` and then create/edit the file `/etc/systemd/journald.conf.d/sdcard.conf`. Copy the following to the file and save it:

    [Journal]
    Storage=volatile
    RuntimeMaxUse=30M

Using `eatmydata`:  
Install the package `libeatmydata`.  
Type `eatmydata *PROGRAM*` where program is the name of the program, to prevent the program from writing files to disk. **Do not use this on a program that want to be able to write data, just use it to prevent unnecessary writing!**

To find out what programs are frequently writing to the disk, use the `iotop` package.

### Installing development basics
Use `pacman -S base-devel git` to install a bunch of software required for developing and compiling.

## Installing and configuring a graphical environment (dwm, st and dmenu)
Use `pacman -S xorg-server xorg-xinit libx11 libxinerama libxft webkit2gtk` to install the required software to compile dwm and st.
Use `git clone https://git.suckless.org/dwm`, `git clone https://git.suckless.org/st` and `git clone https://git.suckless.org/st` to clone the source repositories.  
Navigate to your home directory, create a file called `.xinitrc` and edit:  

```
exec dwm
```

this will run dwm at startup.

Now let's compile:  
`cd st` and `sudo make clean install`  
`cd ../dwm` `sudo make clean install`  
`cd ../dmenu sudo make clean install`
To configure dwm to use st for spawning shell commands, edit `dwm/config.def.h` (around l.51 probably):

```
#define SHCMD(cmd) { .v = (const char*[]){ "/usr/local/bin/st", "-c", cmd, NULL } }

```

and then run `sudo cp dwm/config.def.h dwm/config.h`, to copy the changes.  
Then recompile using `sudo make clean install`.

*(Your path to st **may** deviate from mine, to find out, run `which st`.)*

Now use `cp /etc/skel/.bash_profile ~/.bash_profile` to copy the skeleton-file to your home directory, then, at the end of the file, add `startx`.

***Reboot***

Your should now be booting into dwm after entering your credentials.

**Note:** It is recommended to now move the `st`, `dwm` and `dmenu` directories to `/usr/local/src/`.

### Setting up the environment and installing additional software

#### Changing the mod-key
To use `<Super>` instead of `<Alt>`, edit `dwm/config.def.h`, changing the following:  
From: `#define MODKEY Mod1Mask` to `#define MODKEY Mod4Mask`  
To apply the changes, use `sudo cp dwm/config.def.h dwm/config.h`.  
Then recompile using `sudo make clean install`.

#### Keyboard layout
X and arch do **not** share a keyboard layout. Because of that, you now need to do that for X:  
To temporarily set the layout, you can use `setxkbmap LAYOUT`, where LAYOUT is your layout (for example `us` or `de`).  
To permanently set the layout, either manually edit `/etc/X11/xorg.conf.d/00-keyboard.conf`, according to this wiki page:  
https://wiki.archlinux.org/title/Xorg/Keyboard_configuration  
or use `localectl set-x11-keymap LAYOUT` to generate the contents of that file automatically.

#### Xrandr
Install `xorg-xrandr` using pacman.  
Use `xrandr` to set the screen resolution if required.

#### Audio
Install `alsa-utils` using pacman.
Run `alsamixer` to configure.

#### Web browser
Using pacman, install `lynx` and `qutebroser`.
While `qutebrowser` will serve as the main browser, featuring full web graphics and images, `lynx`, a terminal/text-only browser, is there for situations where bandwidth or power is limited.

#### Wallpaper and image-viewer
Using pacman, install `feh`. `feh` is both an image viewer, and a wallpaper manger for your desktop.  
Use the following command to set your wallpaper: `feh --bg-scale /path/to/image.file`.  
After running that command, a file called `.fehbg` will have been generated in your home directory.  
To automatically set your wallpaper on startup, edit `.xinitrc`, appending:

```
~/.fehbg &
```
### Additional modifications to dwm
To learn how to patch and configure dwm yourself, watch this video (series): https://www.youtube.com/watch?v=DDJu3qlxW-Y  
Alternatively, download my already modified version of `dwm` from this repository.

## Resolving Issues
### **SSL certificate problem: certificate is not yet valid** when using `git clone` on a `https` adress:
Most likely there is an issue with the system time. This subject is prone to issues, as the Raspberry Pi 3 does not have a hardware clock, and relies fully on NTP to function properly.  
To check the date/time, run `date`. To check ntp configuration, run `timedatectl status` and `timedatectl timesync-status`.  
Sometimes, `systemd-networkd` has to be restarted using `systemctl restart` in order for a connection to an NTP server to happen. Visit the **Setting the time** section.

## Unclear
### Time issue
*Starting ```systemctl start systemd-timesyncd.service``` just worked at some point?? After that ```timedatectl set-ntp true``` to enable time sync over internet*.
### pacman key issue
*Seemingly had to do ```pacman-key --init``` and ```pacman-key --populate archlinuxarm``` again after setting time?*

# To research:
With the default configuration, the Raspberry Pi uses HDMI video if a HDMI monitor is connected. Otherwise, it uses analog TV-Out (also known as composite output or RCA) To turn the HDMI or analog TV-Out on or off, have a look at /opt/vc/bin/tvservice https://archlinuxarm.org/platforms/armv6/raspberry-pi

## Sources:
- https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3
- https://www.raspberrypi.org/forums/viewtopic.php?t=90337
- https://wiki.archlinux.org/title/systemd
- https://wiki.archlinux.org/title/NetworkManager
- https://askubuntu.com/questions/795226/how-to-list-all-enabled-services-from-systemctl
- https://wiki.archlinux.org/title/installation_guide
- https://wiki.archlinux.org/title/Install_Arch_Linux_on_a_removable_medium
- https://wiki.archlinux.org/title/Systemd/Journal
- https://wiki.archlinux.org/title/Improving_performance#Reduce_disk_reads/writes
- https://wiki.archlinux.org/title/General_recommendations
- https://www.youtube.com/watch?v=jD8BtmMK0do
- https://www.linux.org.ru/forum/general/16427865
- https://wiki.archlinux.org/title/bash
- https://archlinuxarm.org/platforms/armv6/raspberry-pi
- https://wiki.archlinux.org/title/feh
- https://youtu.be/3QA0TdnE4IU
