# Success-with-Nvidia-545-drivers-in-Ubuntu-22.04-LTS
There is a lot of scattered information on how to succeed with Nvidia GPUs and Ubuntu 22.04. This is what worked for us with a 3070 GPU. 
The resulting happy install is capable of supporting advanced graphics like Unreal Engine 5 as well as AI and ML libraries on the same machine.


Making a happy Nvidia driver and CUDA toolkit for Ubuntu 22.04 LTS going into Q1 2024


**Preamble**

I’m between jobs and had the time to write this and test it. Skip the like-and-subscribe BS. There’s no ads in here either. Just hire me!!! 

Chances are you’ve heard about the rapid pace of AI and how Linux is at the heart of it. You will need hardware and software upgrades both to make it work. Configuring both can be a bit of a challenge for some. The software packages are really just starting to get their sea legs.

If you have a new Nvidia 3060, 3070, or 3080 graphics card and run Ubuntu Linux 22 you’ve likely had a tough time between 2022-2023. There is a LOT of conflicting information in November 2023 on what libraries to install, from where, in what order, and for what bugs that pop up. What works for a 3060 or 3080 doesn’t work for the 3070 8GB that I have. They’re good bang-for-the-buck cards but they all need something a little different. Ubuntu 20 and 22 both act different, as well. 

The Ubuntu libraries and repos are currently a little out of sync. Nvidia drivers need the right cuda toolkit to be happy. The pattern and debug tools listed here will help you along the way. Other professionals will ask you about the results from the common diagnostic tools and logs listed here, too. The bugs listed are common but solutions are scattered. Weeks of singular effort was extended to research and test all of this. It deserves a good and thorough write-up in one spot. 


**Situation**

The range of Nvidia drivers from 470, 515, 525, 535, 545---I tested all of them on my Ubuntu 22.04 LTS install with one 3070 8GB graphics card. Most of them puked. The correct CUDA toolkit version changes according to which driver you use on which card. The Nvidia driver appears to need the CUDA toolkit to function and for you to build onto. Using the GPU in your data science code requires libraries that talk directly to it. The CUDA toolkit is the highway for those libraries. Otherwise you just have a nice gaming rig. This lets you have both! The catch is you can’t cut corners to get there.

As of this writing in November 2023, ubuntu-cuda-toolkit is on version 11.5. I needed version 12.3 from Nvidia to make the new 545 driver work. That’s a bit of a curve ball for newbies. Ubuntu’s repos just haven’t caught up yet and it’s nobody’s fault. This should be a more straight forward install in a few months and I hope to help that along.

Cuda 10-11.4 is recommended by older posts. Ubuntu 22 only partially works with those later drivers. 470 is supposedly more stable but that crashes in Ubuntu 22 as well. 545 finally works. The extra libraries and configuration shown here were the real challenges to piece together.

If you’ve gone around in circles with this until now, these steps should be more straight forward. GPUS need more power and you will end up adding a power supply and a cooler for reliability and stability. Do it or you’ll be sorry. If you have a gaming rig then you likely updated these things already. Making them work smoother in Linux is the point of all these words.


(Bookmark or fork this repo. There’s a few reboots involved.)

**Extra hardware**

1. Get a chip cooler. Your box will thank you. $50.
https://www.amazon.com/Cooler-Master-Silencio-Anodized-Gun-Metal/dp/B07H25DYM3/ref=sr_1_3?crid=3PC8F7KF6MVV&keywords=amd%2Bchip%2Bcooler%2Bhyper%2B212%2Bblack&qid=1700886332&s=electronics&sprefix=amd%2Bchip%2Bcooler%2Bhyper%2B212%2Bblack%2Celectronics%2C87&sr=1-3&th=1

2. Get a good gold rated 850W power supply, like this. If a cap blows on a cheaper one you’ll cook your whole tower and your expensive card. $100
https://www.amazon.com/dp/B0BF3YF9KY?psc=1&ref=ppx_yo2ov_dt_b_product_details

Install the above. I had to cut a hole in the side of my A10 ASUS case for the cooler to fit.


**Prep the BIOS**

1.) Enable discreet graphics. Turn off your integrated or Intel graphics card. Nvidia stuff hates them.

2.) Turn off power saver features. They can mess with the GPU causing some fade and instability from low power. AI, ML, and advanced graphics use more power and need more cooling because of that. If you decide to overclock anything later on, you’ll be turning these functions off anyway because that also takes more power. Hot rods drink expensive fuel and this is no different. You are tuning and prepping a hot rod of a computer. Treat it carefully as such.

3.) Blacklist your nouveau driver. Nouveau drivers can clash with Nvidia and fight over control. Bad juju there.

$ sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
$ sudo bash -c "echo options nouveau modeset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
$ cat /etc/modprobe.d/blacklist-nvidia-nouveau.conf

blacklist nouveau
options nouveau modeset=0
4.) $ sudo update-initramfs -u

5.) update kernel headers.
$ sudo apt-get --reinstall install linux-headers-`uname -r`

6.) Update GRUB just to be safe.
$ sudo update-grub

7.) sudo reboot
Installation

**Clean and prep old libraries in a terminal**

1.) $ sudo apt-get remove nvidia-cuda-toolkit

2. ) $ sudo apt-get --purge remove "*cud*" "*cublas*" "*cufft*" "*cufile*" "*curand*"  "*cusolver*" "*cusparse*" "*gds-tools*" "*npp*" "*nvjpeg*" "nsight*" "*nvvm*"

3.) $ sudo apt-get --purge remove "*nvidia*" "libxnvctrl*"


**Libraries to install**

4. $ sudo apt install lightdm inxi nvidia-settings appmenu-gtk3-module nvidia-settings nvidia-smi nvidia-xconfig

(Special note on lightdm. Ubuntu 22 I think defaults to the GDM3 X Display manager. Nvidia drivers HATE it. Your driver setup will work only once before corrupting. You will see screen artifacts, “melting” of pixels, and what looks like API balloons during the boot up log before graphics have even loaded. Stay away from GDM3 for now.)

5. $ sudo ubuntu-drivers autoinstall
(Special note here: My install grabbed the 545 drivers automatically. Lots of blogs will tell you to do this graphically in the Ubuntu Software Center. Others tell you to install specific nvidia drivers like the 510, 515, 525, etc. That may work for you, but it didn’t work for me. I couldn’t follow anything empirically or literally with this and have it work.)

6. https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=runfile_local 

(adjust specs there as needed for your installation.)

$ sudo sh cuda_12.3.1_545.23.08_linux.run

$ sudo nvidia-xconfig

$ sudo reboot



**Diagnostic results. Bugs explained**

$ nvidia-settings
This should bring up the GUI to check things out with your GPU that the system can now hopefully see.

$ nvidia-smi

Fri Nov 24 23:28:37 2023       
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 545.29.02              Driver Version: 545.29.02    CUDA Version: 12.3     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce RTX 3070        Off | 00000000:01:00.0  On |                  N/A |
| 54%   50C    P8              21W / 240W |    285MiB /  8192MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A      2269      G   /usr/lib/xorg/Xorg                          239MiB |
|    0   N/A  N/A     31882      G   /usr/bin/gnome-shell                         38MiB |
+---------------------------------------------------------------------------------------+


$ nvcc -V
output:
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2023 NVIDIA Corporation
Built on Fri_Nov__3_17:16:49_PDT_2023
Cuda compilation tools, release 12.3, V12.3.103
Build cuda_12.3.r12.3/compiler.33492891_0

If the above library isn’t installed or the driver is not properly installed, the above won’t return any kind of value.


$ dkms status
output:
nvidia/545.29.02, 6.2.0-36-generic, x86_64: installed
nvidia/545.29.02, 6.2.0-37-generic, x86_64: installed

openrazer-driver/3.2.0, 5.15.0-89-generic, x86_64: installed
openrazer-driver/3.2.0, 6.2.0-36-generic, x86_64: installed
openrazer-driver/3.2.0, 6.2.0-37-generic, x86_64: installed
v4l2loopback/0.12.7, 5.15.0-89-generic, x86_64: installed
v4l2loopback/0.12.7, 6.2.0-36-generic, x86_64: installed
v4l2loopback/0.12.7, 6.2.0-37-generic, x86_64: installed
virtualbox/6.1.38, 5.15.0-89-generic, x86_64: installed
virtualbox/6.1.38, 6.2.0-36-generic, x86_64: installed
virtualbox/6.1.38, 6.2.0-37-generic, x86_64: installed


$ sudo prime-select nvidia
Error: no integrated GPU detected.

The above command should output this error if you’ve disabled onboard graphics in the BIOS listed above.

$ grep nvidia /etc/modprobe.d/* /lib/modprobe.d/*
/etc/modprobe.d/blacklist-framebuffer.conf:blacklist nvidiafb
/etc/modprobe.d/nvidia-graphics-drivers-kms.conf:# This file was generated by nvidia-driver-545
/etc/modprobe.d/nvidia-graphics-drivers-kms.conf:options nvidia-drm modeset=1
/etc/modprobe.d/nvidia-installer-disable-nouveau.conf:# generated by nvidia-installer
/lib/modprobe.d/nvidia-installer-disable-nouveau.conf:# generated by nvidia-installer
/lib/modprobe.d/nvidia-kms.conf:# This file was generated by nvidia-prime
/lib/modprobe.d/nvidia-kms.conf:options nvidia-drm modeset=1

The above command will show you if nvidia is blacklisted anywhere. The nvidiafb blacklist should be there but no other nvidia blacklisting as of this writing.

$ grep nouveau /etc/modprobe.d/* /lib/modprobe.d/*
/etc/modprobe.d/blacklist-nouveau.conf:blacklist nouveau
/etc/modprobe.d/blacklist-nouveau.conf:options nouveau modeset=0
/etc/modprobe.d/blacklist-nvidia-nouveau.conf:blacklist nouveau
/etc/modprobe.d/blacklist-nvidia-nouveau.conf:options nouveau modeset=0
/etc/modprobe.d/nvidia-installer-disable-nouveau.conf:blacklist nouveau
/etc/modprobe.d/nvidia-installer-disable-nouveau.conf:options nouveau modeset=0
/lib/modprobe.d/nvidia-graphics-drivers.conf:blacklist nouveau
/lib/modprobe.d/nvidia-graphics-drivers.conf:blacklist lbm-nouveau
/lib/modprobe.d/nvidia-graphics-drivers.conf:alias nouveau off
/lib/modprobe.d/nvidia-graphics-drivers.conf:alias lbm-nouveau off
/lib/modprobe.d/nvidia-installer-disable-nouveau.conf:blacklist nouveau
/lib/modprobe.d/nvidia-installer-disable-nouveau.conf:options nouveau modeset=0

You want to have Nouveau drivers blacklisted. The above command shows that.


$ sudo update-grub

Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.2.0-37-generic
Found initrd image: /boot/initrd.img-6.2.0-37-generic
Found linux image: /boot/vmlinuz-6.2.0-36-generic
Found initrd image: /boot/initrd.img-6.2.0-36-generic
Memtest86+ needs a 16-bit boot, that is not available on EFI, exiting
Warning: os-prober will be executed to detect other bootable partitions.
Its output will be used to detect bootable binaries on them and create new boot entries.
Adding boot menu entry for UEFI Firmware Settings ...
done

The above is more for good house keeping.


Log files

You’ll need log file output if anything goes sideways. This document is also written to help you communicate with others effectively. Tech support or other enthusiasts will help you on StackOverflow, Ubuntu, or Nvidia forums. These log file outputs will help you both diagnose any problems.

https://forums.developer.nvidia.com/t/if-you-have-a-problem-please-read-this-first/27131




