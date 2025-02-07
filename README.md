# Arch Linux Post Installation (my unofficial) Manual 

One day ago from writing this document, I made Arch Linux my daily driver OS. Being a gamer at the same time, I opted to tweak Arch both as a workstation and my gaming rig. Since I encoutnered a number of things that I needed to tweak to achieve amazing naming and working performance, I thought I'd share this info in case it comes in handy to anyone.  

## 1. My Current Specs and Preconditions

```text
                   -`                 
                  .o+`                 taner@homepc
                 `ooo/                 OS: Arch Linux 
                `+oooo:                Kernel: x86_64 Linux 6.13.1-arch1-1
               `+oooooo:               Uptime: 3m
               -+oooooo+:              Packages: 905
             `/:-:++oooo+:             Shell: zsh 5.9
            `/++++/+++++++:            Resolution: 3440x1440
           `/++++++++++++++:           DE: KDE
          `/+++ooooooooooooo/`         WM: KWin
         ./ooosssso++osssssso+`        GTK Theme: Breeze-Dark [GTK2], Breeze [GTK3]
        .oossssso-````/ossssss+`       Icon Theme: breeze-dark
       -osssssso.      :ssssssso.      Disk: 222G / 2.8T (9%)
      :osssssss/        osssso+++.     CPU: AMD Ryzen 7 7800X3D 8-Core @ 16x 5.05GHz
     /ossssssss/        +ssssooo/-     GPU: NVIDIA GeForce RTX 4080 SUPER
   `/ossssso+/:-        -:/+osssso+-   RAM: 4375MiB / 31187MiB
  `+sso+:-`                 `.-/+oso: 
 `++:.                           `-/+/
 .`                                 `/
```

### Preconditions

When installing arch using `archinstaller` command  from the shell i make sure to set the following precoditions(for theyre the best).

- My Arch installation uses a separate partition for /home
- I use the SDDM display manager for the login welcome screen thingie. It also works with Wayland.
- I use KDE Plasma as a desktop environment with `Wayland`. Its amazing.
- I use `pipewire` as the audio driver
- I user `GRUB` as my bootloader
- I chose to use Nvidia proprietary driver (not the open source kernel header stuff which are also from Nvidia and supposed to work amazingly)

## 2. Installing yay

One of the most used AUR package managers is `yay`. Explaining AUR is beyond the scope of this text, and besides i know what it is :) So we run the following steps:

- `sudo pacman -Syu` # to update package list
- `sudo pacman -S --needed base-devel git` # to install git and prerequisites for yay
- `cd ~` # or wherever you want to keep the yay repo
- `git clone https://aur.archlinux.org/yay.git` # to clone yay
- `cd yay` 
- `makepkg -si` # to install it
  
After installation is done we can check `yay --version` to see that everything is installed

## 3. Installing Media Codecs

Pretty straightforward. You might wanna listen to stuff or watch stuff. If so:  
  
`yay -S gst-libav gst-plugins-base gst-plugins-good gst-plugins-bad gst-plugins-ugly gstreamer-vaapi x265 x264 lame`

## 4. Cleaning Package Cache

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

## 5. Gaming Related Stuff and Optimizations

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

```

## 6. Befriending Wayland and NVIDIA

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
