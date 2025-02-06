# Arch Linux Post Installation (my unofficial) Manual 

One day ago from writing this document, I made Arch Linux my daily driver OS. Being a gamer at the same time, I opted to tweak Arch both as a workstation and my gaming rig. Since I encoutnered a number of things that I needed to tweak to achieve amazing naming and working performance, I thought I'd share this info in case it comes in handy to anyone.  

## My Current Specs

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
