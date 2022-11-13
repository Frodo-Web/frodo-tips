# Building and Installing a FreeBSD Kernel
### From FreeBSD documentation: <br>
 Building a custom kernel is often a rite of passage for advanced BSD users. This process, while time consuming, can provide benefits to the FreeBSD system. Unlike the GENERIC kernel, which must support a wide range of hardware, a custom kernel can be stripped down to only provide support for that computer’s hardware. This has a number of benefits, such as:
- Faster boot time. Since the kernel will only probe the hardware on the system, the time it takes the system to boot can decrease.
- Lower memory usage. A custom kernel often uses less memory than the GENERIC kernel by omitting unused features and device drivers. This is important because the kernel code remains resident in physical memory at all times, preventing that memory from being used by applications. For this reason, a custom kernel is useful on a system with a small amount of RAM.
- Additional hardware support. A custom kernel can add support for devices which are not present in the GENERIC kernel.
## 1. Get sources
You need to get the sources of your current release, otherwise some drivers and programs won't work. <br>
Find out your current release:
````
$ uname -r
> 13.1-RELEASE-p3
````
Select this release by using --branch option and put it to /usr/src, make sure /usr/src is empty before doing this:
````
# git clone --branch releng/13.1 https://git.FreeBSD.org/src.git /usr/src
````
You can view all available branches here - [https://cgit.freebsd.org/src](https://cgit.freebsd.org/src) <br>
You can update the sources any time by using these commands:
````
# cd /usr/src
# git pull --rebase
````
## 2. Make changes to kernel configuration
Go to the conf directory matching your system architecture, it's amd64 for me:
````
# cd /usr/src/sys/amd64/conf
````
Create your own config just by copying GENERIC, name it what you want:
````
# cp GENERIC SEGA
````
Make some changes to it, for example, I will rename ident GENERIC to SEGA:
````
# nvim SEGA
# cat SEGA | grep ident 
>> ident SEGA
````
You can comment out devices and options you won't need in your kernel configuration, like ths:
````
...
#device   rl      # ReakTek 8129/8139
#device   sge     # Silicon Integrated Systems SiS190/191
...
````
These commands can help to learn more about your system:
````
// Lists all PCI devices, whether FreeBSD has a driver for it or not.
// If you see something like noneX@pciX:… at the beginning of a line, then no driver is attached to this device.
pciconf -lv

// Lists all USB devices that are currently attached (whether supported by a specific driver or not):
usbconfig

// Review the kernel’s boot messages:
dmesg

// Include other output that has been printed to the system console. 
// Note that the dmesg buffer is circular, so it is overwritten after a while, and the oldest entries get lost
dmesg -a

// In this case you can look at the file /var/run/dmesg.boot that contains a snapshot of the buffer right after the boot process has finished.
cat /var/run/dmesg.boot

// Lists the audio devices that have been found on your system. 
// The list is dynamic, i.e. when you plug in a USB audio device, it will appear in that list.
// The one marked as “default” will be used by most software by default, unless you specify a different device.
cat /dev/sndstat
````
## 3. Build and install the kernel
Go to /usr/src and provide just the config filename to KERNCONF argument:
````
# cd /usr/src
# make buildkernel KERNCONF=SEGA
````
When building is finished, install the kernel to your system:
````
# make installkernel KERNCONF=SEGA
````
Then reboot:
````
# reboot
````
To make sure you were boot with your new kernel, use these commands:
````
$ uname -v
>> FreeBSD 13.1-RELEASE-p3 releng/13.1-n250169-c3c13035ef2 SEGA
````
If you replaced "ident GENERIC" with yours:
````
$ sysctl kern.conftxt | grep ident
>> ident   SEGA
````
You can also view all the options and devices supported left:
````
$ sysctl kern.conftxt 
````
## 4. If you can't boot with new kernel / Something went wrong
On first boot screen, when you see FreeBSD boot menu:
FreeBSD <br>
Welcome to FreeBSD <br>
1. Boot Multi user [Enter]
2. Boot Single user
3. Escape to loader prompt
4. Reboot
5. Cons: Video
Options:
6. Kernel: default/kernel (1 of 2)
7. Boot options

You can press 6 to switch to your old (previouse) kernel:
````
6. Kernel: kernel.old (2 of 2)
````
When you will boot you can restore your previouse kernel back like so:
````
# mv /boot/kernel /boot/kernel.bad
# mv /boot/kernel.old /boot/kernel
````
## 5. Make sure everything works
Check your devices, internet connection - both Ethernet and Wi-fi, hard drive, ssd, programs, browser.
If everything works, you can do cleanup to free some disk space:
````
# cd /usr/src
# make clean
````
