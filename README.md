# Installation Guide
BTW I use arch


### ** issue **
If you ever run into an issue about keyring is corrupted use this 
```sudo pacman -S archlinux-keyring```

cant connect to wifi using networkmanager then use iwd (iwctl)
```h[ttps://www.youtube.com/watch?v=-Dg4WjVu_E0&t=117s](https://www.youtube.com/watch?v=hnwoek_4MXQ)```


## Flashing the arch iso to a flash drive
Flash the arch iso to a flash drive. [Here is the page to the arch .iso file , scroll down and click a link that's under your country. Then download the file that ends with **.iso**](https://archlinux.org/download/)  You can use rufus to flash the iso to the flash drive if you're on windows.

Boot from the newly flashed drive (on a lenovo laptop constantly press F12 on start up and go to the newly flashed flash drive).

## Setting keyboard
type ```loadkeys us``` in the terminal.

## Connecting to the internet
type ```iwctl```

You'll be in the [iwd] enviroment .

Use ```device list``` to get the name of your wifi device name.  _Example of wifi device name: wlan0_

Get the name of the network you want to connect to. **MAKE SURE YOU TYPE YOUR ACUTAL WIFI DEVICE NAME AND NOT "your_device_name"**
```
station your_device_name scan 
station your_device_name get-networks
```
_If you can't see the wifi networks then do_
```
rfkill unblock all
```
_If that doesn't work do_ ```dchpcd```

Connect to the internet. Type your wifi password in. 
```
station your_device_name connect your_network_name
```

Check if you're connected to the wifi.
```
station your_device_name show
```
Nice you're connected to the internet now.
Exit by 
```
exit
```
Now you're back to the arch enviroment.

## Partitioning the disk
If you have a nvme (M.2) you're going to 
```
cfdisk /dev/nvme0n1
```
You can do ```lsblk``` to see if you dont know. You'll see nvme0n1 if you have a nvme or sda if you have a SSD.

If you just have a SSD do
```
cfdisk
```

If it asks you 
|Select label type|
|-----------------|
|       gpt       | 
|       dos       | 
|       sgi       | 
|       sun       | 

Select **gpt**. Some of you may not see that and that is okay.

### Creating the boot partition

If there are devices already there _example: /dev/sda1 ,/dev/sda2_ you want to delete all the devices. So hover over the device and hit enter on delete then new. You can move with the arrow keys. 

Now you should see device named ```Free space```. Hovering over that hit enter on ```[New]``` and delete the original parition size and type ```100M``` hit enter. That is your boot partition. If it asks you ```primary``` or the other thing hit enter on ```primary```.

This should be called /dev/sda1

### Creating the virtual memory partition
Move down to the ```Free space```. Hit enter on ```[New]```. Erase the partition size and type in the size of your physical ram size times two. _Example: physical ram I have is 16GB, then set the virtual memory partition to 32GB_.

This should be /dev/sda2

### Partitioning the rest of your disk
Hover over ```Free space``` hit enter on ```[New]``` hit enter again.

Move over to ```Write``` hit enter. It'll ask you ```Are you sure you want to write the partition table to disk?``` type ```yes``` and hit enter. Go to ```[Quit]``` and hit enter. Clear the screen by holding CTRL and "L".


## Format the partitions

Display block devices
```lsblk```

### Format the root partition
Look at the output of ```lsblk``` and see which one has the largest SIZE. It should be ```sda3```. 
```
mkfs.ext4 /dev/sda3
```
### Format the boot partition
```
mkfs.fat -F 32 /dev/sda1
```
### Format the swap partition
```
mkswap /dev/sda2
```

## Mounting partitions

### Mounting the root partition
```
mount /dev/sda3 /mnt
```
### Mounting the boot partition
```
mkdir -p /mnt/boot/efi
```
### Mounting the sda1
```
mount /dev/sda1 /mnt/boot/efi
```
### Turn on swapon
```
swapon /dev/sda2
```

## Installing necessary packages
First do this
```
pacman -Sy archlinux-keyring
```
Do this if you only want a base arch install.
```
pacstrap /mnt base linux linux-firmware sof-fimware base-devel grub efibootmgr vim nano networkmanager  
```

Don't do this because this is the packages I want.
```
pacstrap /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr vim networkmanager dhcpcd reflector iwd alacritty dnsmasq polybar nitrogen  neofetch ranger dmenu
```

## Generating the file system tab
To show the file systems that are mounted
```
genfstab /mnt
```
To move this to out of the terminal and into a file in our disk
```
genfstab /mnt > /mnt/etc/fstab
```
To check you did it right do. The output should be the same as ```genfstab /mnt```.
```
cat /mnt/etc/fstab
```

## Entering to the installed system
To change root
```
arch-chroot /mnt
```
### Setting the timezone
You can change the country and sub zone. _Example: America/New_York instead of country/subzone_
```
ln -sf /usr/share/zoneinfo/country/subzone /etc/localtime
```
To check if you have the correct country and subzone run
```
date
```
To sync the system clock
```
hwclock --systohc
```

## Localization
You're going to edit the locale.gen file. I'm going to use vim as the text editor and It's okay if you don't know how to use vim I will help you along the way. You will have do exactly as I say if you want to use vim and don't know how to use it. If you mess up after typing ```vim /etc/locale.gen``` press ```ESC``` on your keyboard then type ```:q!``` and hit enter. This will make you leave the text editor without making any changes. So if you mess up do that and go back into the file using vim. If you're more comfortable with nano use it.
```
vim /etc/locale.gen
```
Now we'll navigate to "#en_US.UTF-8 UTF-8", we do this by typing
```
/#en_US.UTF-8 UTF-8
```
Then hit enter and now press ```s``` This will remove the "#" that is in front of ```US.UTF-8 UTF-8```

To save and exit press ```ESC``` on the keyboard and type ``:wq`` this will save the file and quit out of vim.

To generate locale type
```
locale-gen
```
To specify the locale, create a file by
```
vim /etc/locale.conf    
```
Press ```i``` this will get you into insert mode so you can type stuff in the file and type ```LANG=en_US.UTF-8```. Press ```ESC``` on the keyboard to go into normal mode.To save the file and leave type ```:wq```

## Declaring the host name 
Create a file ```hostname``` in the /etc directory
```vim /etc/hostname```
Type in the name you want. Then save and exit vim.  

## Creating the root password
```passwd``` 

# Adding a user
``` useradd -m -G wheel -s /bin/bash type_a_username```

To set a user password
```passwd type_a_username``

# Creating sudo
```EDITOR=vim visudo```
Find ```# %wheel ALL=(ALL) ALL``` and uncomment the ```#``` then save and quit

## Enabling networkmanager
```systemctl enable NetworkManager```

## Setting up grub
```grub-install /dev/sda```
config grub
```grub-mkconfig -o /boot/grub/grub.cfg```
exit by ```exit```

## Exiting archiso
```umount -a```
```reboot```

## Update your arch every biweekly 
use ```sudo pacman -Syu```

if there's issue about invalid something do this
```sudo pacman -Sy archlinux-keyring && sudo pacman -Su```



## Setting up Awesome WM



















































