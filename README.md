# Arch Linux Installation and Post-Installation (my unofficial) Manual

Last Update: 2025-03-06

I made Arch Linux my daily driver OS. Being a gamer at the same time, I opted to tweak Arch both as a workstation and my gaming rig. Since I encoutnered a number of things that I needed to tweak to achieve amazing naming and working performance, I thought I'd share this info in case it comes in handy to anyone.  
  
This is NOT an Arch instalation guide. Arch Linux’s flexibility is one of its greatest strengths, but that freedom also comes with the responsibility to build a setup that works best for you. There are a lot of those online and are easily searchable. It is basically a todo list for me of things to remember when tweaking my installation for gaming. Taking the time to complete essential post-installation steps is an investment in a smooth and personalized experience, especially for gaming. I wanted to share my steps here.  
  
**UPDATE** I also decided to add the installation steps for my own reference since i really enjoyed the installation process itself.  
  
So without further delay...  

## My Current Specs and Preconditions

When installing arch using `archinstaller` command  from the shell i make sure to set the following precoditions (for theyre the best and reflect the above stuff in the INstallation section).

- My Arch installation uses a separate partition for /home
- I use the SDDM display manager for the login welcome screen thingie. It also works with Wayland.
- I use KDE Plasma as a desktop environment with `Wayland`. Its amazing.
- I use `pipewire` as the audio driver
- I user `GRUB` as my bootloader
- I chose to use zen kernel because its more optimized 

I use Arch as my daily driver so I dont treat this OS as a secondary "whatever" dual-booted thing, but as my goto OS that needs to work with everything including gaming, work and entertainment. So the installation process is catered to that. These are my specs:

```text
                   -`                    tanerius@archos 
                  .o+`                   --------------- 
                 `ooo/                   OS: Arch Linux x86_64 
                `+oooo:                  Host: MS-7E12 1.0 
               `+oooooo:                 Kernel: 6.13.5-zen1-1-zen 
               -+oooooo+:                Uptime: 22 mins 
             `/:-:++oooo+:               Packages: 992 (pacman), 9 (flatpak) 
            `/++++/+++++++:              Shell: zsh 5.9 
           `/++++++++++++++:             Resolution: 3440x1440 
          `/+++ooooooooooooo/`           DE: Plasma 6.3.2 
         ./ooosssso++osssssso+`          WM: kwin 
        .oossssso-````/ossssss+`         Theme: Adwaita [GTK3] 
       -osssssso.      :ssssssso.        Icons: Adwaita [GTK3] 
      :osssssss/        osssso+++.       Terminal: konsole 
     /ossssssss/        +ssssooo/-       CPU: AMD Ryzen 7 7800X3D (16) @ 5.050GHz 
   `/ossssso+/:-        -:/+osssso+-     GPU: AMD ATI 16:00.0 Raphael 
  `+sso+:-`                 `.-/+oso:    GPU: NVIDIA GeForce RTX 4080 SUPER 
 `++:.                           `-/+/   Memory: 3995MiB / 31182MiB 
 .`                                 `/
```

## Installation

To verify verify the boot mode do the follwoing and make sure the result is 64 (the system is booted in UEFI mode and has a 64-bit x64 UEFI). 
```bash
cat /sys/firmware/efi/fw_platform_size
```
  
To see all the network devices. Also just to make sure you are connected ping a website.
```bash
ip link 
ping -c 3 archlinux.org
```
  
Sync the clock
```bash
timedatectl
```

### Partitioning
  
Partition the disks using `dfisk /dev/<deviceid>`. Here is some useful info for this:
```text
Partition type aliases:
   linux          - 0FC63DAF-8483-4772-8E79-3D69D8477DE4
   swap           - 0657FD6D-A4AB-43C4-84E5-0933C84B4F4F
   home           - 933AC7E1-2EB4-4F13-B844-0E14E2AEF915
   uefi           - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
   raid           - A19D880F-05FC-4D3B-A006-743F0F84911E
   lvm            - E6D6D379-F507-44C2-A23C-238F2A3DF928

Partition type ids (common ones):
   1 EFI System                     C12A7328-F81F-11D2-BA4B-00A0C93EC93B
   2 MBR partition scheme           024DEE41-33E7-11D3-9D69-0008C781F39F
  ...
  19 Linux swap                     0657FD6D-A4AB-43C4-84E5-0933C84B4F4F
  20 Linux filesystem               0FC63DAF-8483-4772-8E79-3D69D8477DE4
  ...
  23 Linux root (x86-64)            4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709
  ...
  41 Linux home                     933AC7E1-2EB4-4F13-B844-0E14E2AEF915
  42 Linux RAID                     A19D880F-05FC-4D3B-A006-743F0F84911E
  43 Linux LVM                      E6D6D379-F507-44C2-A23C-238F2A3DF928
  44 Linux variable data            4D21B016-B534-45C2-A9FB-5C16E091FD2D
  45 Linux temporary data           7EC6F557-3BC5-4ACA-B293-16EF5DF639D1
```
  
Steps and layout for a UEFI with GPT partitioning:
- `/boot` EFI partition at least 1GB (I would today go with 2) - type 1 (format with `# mkfs.fat -F 32 /dev/efi_system_partition`)
- swap partition of 4GB - type 19 (format with `# mkswap /dev/swap_partition`)
- `/root` - type 23 (format with `# mkfs.ext4 /dev/partition_id` )
- `/home` - type 41 (format with `# mkfs.ext4 /dev/partition_id` )
  
### Mounting

Mount the partitions with:
```bash
mount /dev/root_partition /mnt

### Any other mount points like /home .

mount --mkdir /dev/efi_system_partition /mnt/boot
swapon /dev/swap_partition
```

### Installation and Initial Config

- See `/etc/pacman.d/mirrorlist` if you like to edit it. Later we should use `reflector`.
- `pacstrap -K /mnt base linux linux-headers linux-firmware` to install the base system with vanila kernel or
- `pacstrap -K /mnt base linux-zen linux-zen-headers linux-firmware` to install the base system with zen kernel.

Instead of `kernel` which is the vanila kernel we can also install other versions. Today i would go with zen kernel for gaming. Here are the supported ones:
```text
Stable — Vanilla Linux kernel and modules, with a few patches applied.
https://www.kernel.org/ || linux

Hardened — A security-focused Linux kernel applying a set of hardening patches to mitigate kernel and userspace exploits. It also enables more upstream kernel hardening features than linux.
https://github.com/anthraxx/linux-hardened || linux-hardened

Longterm — Long-term support (LTS) Linux kernel and modules with configuration options targeting usage in servers.
https://www.kernel.org/ || linux-lts

Realtime kernel — Maintained by a small group of core developers led by Ingo Molnar. This patch allows nearly all of the kernel to be preempted, with the exception of a few very small regions of code ("raw_spinlock critical regions"). This is done by replacing most kernel spinlocks with mutexes that support priority inheritance, as well as moving all interrupt and software interrupts to kernel threads.
https://wiki.linuxfoundation.org/realtime/start || linux-rt, linux-rt-lts
Note: Real-time kernel support was merged into Linux 6.12

Zen Kernel — Result of a collaborative effort of kernel hackers to provide the best Linux kernel possible for everyday systems. For more details see FAQ and Detailed Feature List.
https://github.com/zen-kernel/zen-kernel || linux-zen
```
  
Generate and check the initial fstab
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
  
Chroot into the new system
```bash
arch-chroot /mnt
```
  
Set timezone and time shizzle
```bash
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
hwclock --systohc
```

### Localization

- Edit `/etc/locale.gen` and enable locales you like (in any case enable `en_US.UTF-8 UTF-8`)
- run `locale-gen` 
- Edit/Create `/etc/locale.conf` and add `LANG=en_US.UTF-8`

### Finalization Checklist

- Do `echo hostname > /etc/hostname` to set the hostname
- Set root password with `passwd`
- Add an editor `pacman -S nano` (or vim if ur crazy!)
- Add network manager or you won't have internet on reboot:
  - Do `pacman -S networkmmanager` to install the network manager
  - Enable it as a service `systemctl enable NetworkManager.service`
  - Enable systemd-resolved `systemctl enable systemd-resolved`
  - Edit/Add whatever you need in `/etc/systemd/resolved.conf` (Normally it will work as is but Google it ... its important)
- Install GRUB and EFI boot manager
  - `pacman -S grub efibootmgr`
- Install the GRUB EFI application grubx64.efi to `/boot/EFI/Arch`. (Arch can actually be any ID specified in `--bootloader-id`)
  - `grub-install --target=x86_64-efi --efi-directory= /boot --bootloader-id=Arch`
- After the installation, the main configuration file `/boot/grub/grub.cfg` needs to be generated
  - `grub-mkconfig -o /boot/grub/grub.cfg` - **IMPORTANT** run this every time after installing or removing a kernel!!
- Add additional custom menu entries if needed (NOT NEEDED but here just to satisfy my dual boot another linux or something)
  - Edit `/etc/grub.d/40_custom`
- Install microcode - DON'T INSTALL BOTH:
  - `pacman -S grub amd-ucode` for AMD CPUs **or**
  - `pacman -S grub intel-ucode` for Intel CPUs
- Install man - this is badass, u never need it but if you do its a lifesaver.
  - `pacman -S man man-db man-pages texinfo`
- Install pipewire for audio and some other audio shizzle
  - `pacman -S pipewire` sound system (will ask for extras install everything probably best)
  - `pacman -S pipewire-docs` to review the documentation
  - `pacman -S wireplumber` session and policy manager for pipewire
- Install nvidia drivers (PICK ONE) - If you have AMD GPU go here https://wiki.archlinux.org/title/Xorg#AMD
  - `pacman -S nvidia` (linux kernel)
  - `pacman -S nvidia-lts` (linux-lts kernel)
  - `pacman -S nvidia-dkms` (any kernel - preferred)
  - `pacman -S nvidia-utils lib32-nvidia-utils` to unstall vulkan support.
- Do `pacman -S neofetch htop` to get some fancy apps to list your system specs
- Edit `/etc/pacman.conf` to your liking
  - Enable simultaneous downloads
  - Enable Colors
  - Enable that pacman thingie by adding `ILoveCandy` to `/etc/pacman.conf` that makes the progress look like pacman :)
  - Maybe enable cleaning the versions cache (google it)
  - If you plan on using 32-bit applications, you will want to enable the multilib repository. (Try not to give these oldies any spotlight though)
- Install and configure `reflector` 
- Install `sudo` and set the text editor for visudo
  - `pacman -S sudo`
  - `export SUDO_EDITOR=nano` (put this someplace where you can source it every login)
  - Do `visudo` and enable all members of sudo group to do sudo!
- Do `groupadd sudo` to create the sudoers group
- Create a non root user and add it to the relevant groups:
  - `useradd -m tanerius` to add the user and create his home folder
  - `usermod -a -G sudo tanerius` to let tanerius issue sudo commands
  - `usermod -a -G video tanerius` seems to be important for video shizzle
  - Add to any other group u need
  - Change the password of the user `passwd tanerius`

## Post Installation

Now starts your installation tweaking and post install steps. From here of everything is more about personal preferences and such. From now of we should not be using `root` user anymore but rather our main non root. Best is to reboot actually. 

### Installing yay

One of the most used AUR package managers is `yay`. Explaining AUR is beyond the scope of this text, and besides i know what it is :) So we run the following steps:

- `sudo pacman -Syu` # to update package list
- `sudo pacman -S --needed base-devel git` # to install git and prerequisites for yay
- `cd ~` # or wherever you want to keep the yay repo
- `git clone https://aur.archlinux.org/yay.git` # to clone yay
- `cd yay` 
- `makepkg -si` # to install it
  
After installation is done we can check `yay --version` to see that everything is installed

### Install KDE and SDDM Display Manager

Enable extras `/etc/environment`

```bash
# for nvidia GPUs
GBM_BACKEND=nvidia-drm 
# for nvidia GPUs
__GLX_VENDOR_LIBRARY_NAME=nvidia

# this is needed for a fresh manual linux
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
LANGUAGE=en_US.UTF-8
```
  
Install the minimal version of KDE Plasma and `sddm` Display Manager

```bash
sudo pacman -S plasma-desktop sddm sddm-kcm qt5-declarative
```
  
Enable the `sddm.service`

```bash
sudo systemctl enable sddm.service
```
  
Configure SDDM bu creating the file `/etc/sddm.conf.d/tanerius.conf`. Instead of tanerius call it what u want. Add the following:

```bash
[General]
# Enable Qt's automatic high-DPI scaling
EnableHiDPI=true

[Theme]
# Current theme name
Current=breeze

# Cursor theme used in the greeter
CursorTheme=breeze_cursors

# Enable display of custom user avatars
EnableAvatars=true

# Theme directory path
ThemeDir=/usr/share/sddm/themes
```
  
Install the apps you will need now to use in KDE:

```bash
# Terminals: a really simple terminal and really fancy terminal
sudo pacman -S alacritty konsole

# Our file manager: dolphin
sudo pacman -S dolphin dolphin-plugins kompare kio-gdrive baloo

# some plugins/additions to dolphin that are useful 
sudo pacman -S kdegraphics-thumbnailers kimageformats qt6-imageformats resvg ffmpeg-thumbs ffmpegthumbs taglib icoutils

# Install KWrite for taking notes 
sudo pacman -S kwrite

# KWallet and utilities to enable storing secrets and whatnots
sudo pacman -S kwallet kwalletmanager kwallet-pam

# Utility to graphically configure refresh rates of the monitor
sudo pacman -S kscreen

# Super useful document viewer
sudo pacman -S okular

# Screen capture (screenshot) utility 
sudo pacman -S spectacle

# An image viewer
sudo pacman -S gwenview

# A print app
sudo pacman -S kolourpaint

# From now lets use yay cuz its cooler and it will install stuff also from AUR where needed

# Flatpak manager for flatpaks
yay -S discover flatpak fwupd

# A calculator cuz i mean you need one
yay -S kalk

# A fancy volume control widget
yay -S plasma-pa kmix

# An image writer to experimet with other images
yay -S isoimagewriter

# An archeiving tool
yay -S ark

# Install a browser - i like chrome but chromium also is OKish
yay -S google-chrome
```
  
This is all i have on my system as i don't need the rest of the KDE Bloat. But a really nice list of all the apps can be found on this link: [https://apps.kde.org/](https://apps.kde.org/)

For more detailed info see.
- [https://wiki.archlinux.org/title/SDDM](https://wiki.archlinux.org/title/SDDM)
- [https://community.kde.org/Distributions/Packaging_Recommendations](https://community.kde.org/Distributions/Packaging_Recommendations)
- [https://wiki.archlinux.org/title/KDE](https://wiki.archlinux.org/title/KDE)

### Make your shell look amazing and do amazing shit

Install `zsh`

```bash
yay -S zsh
```
  
Install `oh-my-zsh` for more swag

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
  
Install the best `oh-my-zsh` plugins:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/MichaelAquilina/zsh-you-should-use.git $ZSH_CUSTOM/plugins/you-should-use

git clone https://github.com/fdellwing/zsh-bat.git $ZSH_CUSTOM/plugins/zsh-bat
```
  
Edit `~/.zshrc` and enable the plugins

```bash
nano ~/.zshrc

# find and replace ZSH_THEME and plugins with the following:
...
ZSH_THEME="bira"
...
plugins=(git zsh-autosuggestions zsh-syntax-highlighting you-should-use zsh-bat)
...
```

### Installing Media Codecs

Pretty straightforward. You might wanna listen to stuff or watch stuff. If so:  
  
`yay -S gst-libav gst-plugins-base gst-plugins-good gst-plugins-bad gst-plugins-ugly gstreamer-vaapi x265 x264 lame`

### Cleaning Package Cache

This will clear the package cache to only keep 1 version after every action. This is good to keep disk usage minimal. Run:
  
```bash
yay -S pacman-contrib
sudo mkdir /etc/pacman.d/hooks
sudo nano /etc/pacman.d/hooks/clean_package_cache.hook
```

In `/etc/pacman.d/hooks/clean_package_cache.hook` write:

```text
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *
[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -rk 2
```

### Gaming Related Stuff and Optimizations

The post install steps are related to gaming. Run the following commands:

```bash
## Install Vulkan Driver (AMD GPU Only)
sudo pacman -S vulkan-radeon lib32-vulkan-radeon

## Install Vulkan Driver (NVIDIA GPU Only)
sudo pacman -S nvidia-utils lib32-nvidia-utils

## Install gaming stuff
sudo pacman -S chromium git steam gamemode mangohud wine-staging

## Install overlay to measure fps and stuff
yay -S --noconfirm goverlay

## Disable file indexer in Dolphin
balooctl6 suspend && balooctl6 disable && balooctl6 purge

## Fonts
yay -S --noconfirm --quiet --needed ttf-ms-win11-auto
sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts-extra ttf-liberation ttf-dejavu ttf-roboto adobe-source-code-pro-fonts ttf-jetbrains-mono

## Libreoffice stable maintenance branch
sudo pacman -S libreoffice-still
```

## Befriending Wayland and NVIDIA

Because I was keen to use Wayland (instead of X11) and having read that Nvidia and Wayland are not good "friends", the first thing to tacke was to make sure they work well together.

### Setting Environment Variables

With NVIDIA’s introduction of GBM (Generic Buffer Management) support, a crucial component of the Linux graphics stack that provides an API for allocating buffers for graphics rendering and display, many compositors have adopted it as their default. So, to force GBM as a backend, we need to set some environment variables.  
  
Edit `/etc/environment` and add the following variables:  
  
```bash
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia
```

### Loading the NVIDIA Modules at System Boot

We want to ensure that the NVIDIA modules are loaded at system boot.
  
Edit `/etc/mkinitcpio.conf` find the `MODULES` section of this file and make sure to add the values ‘nvidia,’ ‘nvidia_modeset,’ ‘nvidia_uvm,’ and ‘nvidia_drm‘. It should look like the following:  
  
```bash
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```  
  
In the same file remove the value `kms` from the `HOOKS` section to avoid including the "nouveau" driver.  
  
After doing this regenerate _initramfs_ by doing:  
  
```bash
sudo mkinitcpio -P
```
  
### Enable Direect Rendering Manager (DRM)

DRM (Direct Rendering Manager) is a subsystem of the Linux kernel responsible for interfacing with GPUs. It provides a framework for graphics drivers to enable direct access to the graphics hardware, which is crucial for performance in rendering tasks, 3D graphics, video playback, and more.
  
We enable DRm by passing the kernel mode setting as a parameter to the Linux kernel during boot. Since i am using GRUB, I edit `/etc/default/grub` and append `nvidia-drm.modeset=1 nvidia_drm.fbdev=1` to the value of the `GRUB_CMDLINE_LINUX_DEFAULT` parameter. 

Additionally, if you’re using the KDE Plasma desktop (like me), it’s important to add `nvidia.NVreg_EnableGpuFirmware=0` to the settings mentioned above. The line in my GRUB config looks like the following:  
  
```bash
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia-drm.modeset=1 nvidia_drm.fbdev=1 nvidia.NVreg_EnableGpuFirmware=0"
```
  
Then, regenerate the GRUB configuration by running:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
  
Finally, restart your PC.

## 7. CPU Microcode Update

```bash
# Install microcode for AMD (for intel instead of amd put intel)
sudo pacman -S amd-ucode
# then update grub config
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
