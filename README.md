# Arch Linux Installation Guide
This page is a complete guide for getting an arch linux virtual machine up and running on a MacBook Pro in **one** *simple* step. There are no guarantees that this guide would work on a Windows machine because I did not try it. 

---

## 1.1) Pre-Installation
### 1.1.1) Acquire a verified image of Arch

I acquired an installation image and needed to verify the signature to ensure I was downloading trusted software. 

Command to verify: `shasum -a 256 /path/to/file`

Output: `17fe2053a114f2002efed53b39f740dd9778f5b689c9467a310b2649a80a6bfd`  
SHA256: `17fe2053a114f2002efed53b39f740dd9778f5b689c9467a310b2649a80a6bfd`

I compared the checksum given by the command to the checksum in the downloads section of the Arch wiki. They were identical so my download is trusted.

### 1.1.2) Open the image on VMware

I created a new virtual machine using the iso file and allocated resources for the VM to be able to operate efficiently. I then booted the VM and it loaded into a terminal:  

### 1.1.3) Verify UEFI Boot

I checked for files efi firmware in the /sys folder to make sure there were files there. If there weren't then it would show that the system was not booted via UEFI.

### 1.1.4) Set Timezone

I set the timezone to be US-Central using command:

```bash
timedatectl set-timezone US/Central
```

### 1.1.5) Partition the Disk

I gave the OS 25 GB to work with on its disk, but I needed to partition this to make it usable:

```bash
cfdisk /dev/sda
```

I then chose GPT for the partitioning type because we are booting from UEFI

Per recommendations found online and accepted as truth, I created 3 partitions. The first being for the EFI boot, the second is for the OS itself and the third is for the SWAP space. The swap space is used when the virtual machine's ram potentially hits maximum utilization. Inactive pages are moved into the swap space.

The EFI partition(/sda1) was formatted to be a FAT32 partition:

```bash
mkfs.fat -F 32 /dev/sda1
```

The filesystem (/sda2) was formatted to be an ext4 partition:

```bash
mkfs.ext4 /dev/sda2
```

The swap needed to be manually created and turned on using:

```bash
mkswap /dev/sda3
swapon /dev/sda3
```

The partitions also needed to be mounted:

```bash
mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot
```

You are almost. Don't worry.

---

## 1.2) Install & Configure Arch

### 1.2.1) Install the base system
```bash
pacstrap -K /mnt base linux linux-firmware
```

The command will initiate the install all of the essential packages for arch.  
Note: This does not mean it will have all the tools and programs that a beginner is used to having. It means it will install all absolutely mission critical packages that arch needs. I found this out the hard way.

### 1.2.2) Setup fstrap

Fstrap is used to inform the OS on how the hard drive partitions should be mounted after being booted

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

### 1.2.3) Change Root

Switch yourself onto the mounted partition for the forseeable future of this installation.

```bash
arch-chroot /mnt 
```

---

## 1.3) Install a text editor

Install a text editor via pacman (I chose nano). If you don't do this now, it will make things very difficult moving forward. 

```bash
pacman -S nano
```

---

## 1.4) Update locale info

I opened the file called *locale.gen* and uncommented *en_US.UTF-8*:

```bash
nano /etc/locale.gen
```

I then ran:

```bash
locale-gen
```

To set the default language system I updated the *locale.conf* file:

```bash
nano /etc/locale.conf
```

I changed the LANG=*en_US.UTF-8* and then saved the file.

I updated the hostname file (/etc/hostname) to include a name for my machine: I named him **archie** :)

---

## 1.5) Enable networking services

This is a crucial step. If you skip this step, you will not have internet access on your machine once you boot it up.

```bash
systemctl enable systemd-networkd
systemctl enable systemd-resolved
```

I also configured the /etc/systemd/network/20-wired.network file to include the configuration for what the arch iso is currently using to provide network access. I used the interface name **ens33** which was found using `ip addr`.

```/etc/systemd/network/20-wired.network
[Match]
Name=ens33

[Network]
DHCP=yes
```

---

## 1.6) Set a password

I set a password using `passwd`

---

## 1.7) Install microcode for Intel processor

```bash
pacman -S intel-ucode
```

---

## 1.8) Install and configure bootloader

I installed grub as the bootloader

```bash
pacman -S grub efibootmgr
```

I installed grub as the bootloader on the efi partition. Some tutorials will not specify the `efi-directory` however I found that it was necessary to include it because the iso was unable to find it otherwise.

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

I generated the main grub config file:

```shell
grub-mkconfig -o /boot/grub/grub.cfg
```

I exited the `/mnt` environment and ran:

```bash
bootctl install --esp-path=/mnt/boot`
```

I rebooted the virtual machine:

```bash
reboot
```

Upon reboot, a blue boot manager screen appeared and I selected boot from GRUB. This completed the install and my arch vm was up and running. 

---

## 1.9) Installing the GUI

I run all of the following code in the Arch CLI:

### 1.9.1) Install Xorg
```bash
pacman -S xorg xorg-server
```

I installed all of the dependencies when prompted as well.

### 1.9.2) Install GNOME
I chose to install the GNOME gui because it has a funny name. 

```bash
pacman -S gnome
```

The installation prompt will ask to install extra packages. Just say yes to them and move on.

### 1.9.3) Start gdm.service
```bash
systemctl start gdm.service
systemctl enable gdm.service
```

Starting the gdm service immediately brings up the GNOME environment. Enabling the service makes it so that the GUI will start on boot. This combination of start/enable will be commonly used for the remainder of the installation. 

---

## 1.10) Add User Accounts

I used the following commands to create two users on the machine.
```bash
useradd sean
passwd *****

useradd codi
passwd GraceHopper1906
passwd -e codi
```

To give both users sudo permissions, I needed to update the `/etc/sudoers` file

```
root ALL=(ALL) ALL
%wheel ALL=(ALL) ALL
```
I then run the `usermod` command for any user that I wish to give sudo permissions to.
```zsh
usermod -aG wheel sean
usermod -aG wheel codi
```
This command adds the users to wheel group. The wheel group has sudo permissions.

---

## 1.11) Configuring the terminal to look cool

### 1.11.1) Install a Different Shell (zsh)

I chose to install zsh because it is what I am most used to on macOS.

```bash
pacman -S zsh
pacman -S zsh-completions

chsh -s /bin/zsh root
```

### 1.11.2) Color Customization
I used a framework called `oh-my-zsh` to make all of my customizations because it allows for me to use pre-made templates and then build off of them instead of starting from scratch.
```zsh
pacman -S git
curl -L http://install.ohmyz.sh | sh
```
I needed git to be able to clone the repository with all of the framework files. I then used `curl` to install the framework. I chose `bira` as my template because it has a good coloring scheme and I like the way it looks.

### 1.11.3) Aliasing
```zsh
alias sshGateway="ssh sysadmin@129.244.245.111"
```
I created an alias to connect to ssh without needing to remember the ip address. Alternatively, I later learn that you can configure the `.ssh/config` file to store hosts that you want to ssh into. For example:
```zsh
Host gateway
	Hostname 129.244.245.111
	User sysadmin
```
The `bira` theme that I am using in my terminal also comes with dozens of pre-installed aliases for a variety of purposes ranging from easier changing of directories to simplifying git commands:
```zsh
g=git
ga='git add'
gam='git am'
...=../..
....=../../..
```
The dot aliases extend the native `cd ..` command to go back to parent directories. These aliases allow for you to go deep backwards into the hierarchy if you want to.

---

## 1.12) SSH
### 1.12.1) Installing SSH
```zsh
pacman -Syu
reboot
pacman -S openssh
systemctl status sshd
systemctl start sshd
systemctl enable sshd
```
### 1.12.2) Configuring SSH
```zsh
systemctl status sshd
systemctl start sshd
systemctl enable sshd
```

### 1.12.3) Connect to gateway
```zsh
ssh sysadmin@129.244.245.111
# asks to store fingerprint of host
# asks for password (suYqs9YuxR43)

```