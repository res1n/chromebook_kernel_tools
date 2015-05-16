
# Installing Arch Linux on Toshiba Chromebook 2

**THIS GUIDE IS NOT YET COMPLETE. IN ITS CURRENT STATE IT SHOULD GET YOU TO A BOOTING ARCH LINUX INSTALLATION WITH A FEW ISSUES. SOLUTIONS TO THESE ISSUES WILL BE ADDED SOON**

This guide details the process I used to install Arch Linux on my Toshiba Chromebook 2.  The guide has been modified from github.com/drsn0w for his baytrail findings.

Currently this guide is only for Linux.

### Note
I was on chromeos 41.0.2272.89 (64 bit) firmware Google_Swanky.5216.238.5
While performing this, so powerwash + reset may be required.

### Known Issues
- Sound does not work
- Suspend does not work
- Kernel Panic on Shutdown / reboot
- Creating Volitile files 2 min delay on boot

## Requirements
- Toshiba Chromebook 2 (others may work, untested)
- Another computer running Linux
- squashfs-utils
- Chrome OS Recovery Media (make this before you start!!!)
- Flash drive larger than 2GB
- Lots of patience.


## Prepare Arch Linux USB Drive
*Commands in this section are to be typed on your Linux computer*

1. Partition your USB flash drive with a GPT partition table and one ext4 partition using all available space.
2. Download the latest Arch Linux ISO image from the Arch Linux website.
3. Using an archive utility, extract the following from the Arch Linux ISO: `arch/x86_64/airootfs.sfs`.
4. `unsquashfs airootfs.sfs`
5. mount the .img file extracted by unsquashfs with mount -o loop *.img yourdir/
6. Copy the files that you mounted to your usb with `cp -R yourdir/* /USB/mount/point`
7. Unmount USB drive

## Prepare Chromebook for booting from Arch Installer
*Commands in this section are to be typed on your Chromebook in Chrome OS*

1. Make sure your Chromebook is in Developer Mode. If you're not sure, it probably isn't. Google how to do this, as it is outside the scope of these instructions. 
2. Download this repo zip or tar.
3. Open Crosh and type `shell`
4. `sudo crossystem dev_boot_signed_only=0`
4. `cd ~/Downloads`
5. Unzip or `tar -xvsf repo.tarball.tar.gz` the downloaded file
6. `cd repo-directory`
7. `cp /usr/share/vboot/devkeys/kernel_data_key.vbprivk ./`
8. `cp /usr/share/vboot/devkeys/kernel.keyblock ./`
9. `bash extract_kernel`
10. **We create two kernels here, one is flashed now, and one after arch is installed plug in your install usb now**
11. `echo "root=/dev/mmcblk0p1 ro rootwait" > config.txt`
12. `bash repack_kernel`
13. `cp repacked2 /USB/mount/point/`
14. `echo "root=/dev/sda1 ro rootwait" > config.txt`
15. `bash repack_kernel`
16. **MAKE SURE THERE ARE NO ERRORS. FLASHING A BAD KERNEL WILL REQUIRE YOU TO RECOVER YOUR CHROMEBOOK AND START FROM THE BEGINNING OF THIS SECTION**
17. `bash flash_kernel`
18. Please note that at this point, your Chromebook will not boot into Chrome OS without being Recovered.
29. `cp -R /lib/modules/3.10.18 /usb/mount/point/usr/lib/modules/`
20. `cp /usr/share/vboot/devkeys/kernel_data_key.vprivk /usb/mount/point`
21. `cp /usr/share/vboot/devkeys/kernel.keyblock /usb/mount/point`
22. Shutdown your Chromebook.

## Boot the installation medium on your Chromebook

1. Press the Power button on your Chromebook.
2. Press Control+D to bypass the Developer Mode warning screen.
3. If all goes well, the Arch installation shell should boot. 
4. If it hangs on "Creating Volatile Files and Directories", wait. After two minutes it will boot.

*You should be at a root shell with the prompt `root@archiso# `. This is the Arch Linux installation prompt.*

## Installing Arch Linux

1. `mount -o remount,rw /`
2. `mkfs.ext4 /dev/mmcblk0p1`
3. `mount /dev/mmcblk0p1 /mnt`
4. Use `wifi-menu` to establish an internet connection. Note that it may take up to three minutes for Arch Linux to recognise the WiFi card, so be patient if wifi-menu doesn't work.
5. Follow the [Arch Linux Beginner's Guide starting at Select a mirror](https://wiki.archlinux.org/index.php/Beginners%27_guide#Select_a_mirror), up until "Install and configure a bootloader". **Do not Install and configure a bootloader.**
6. `exit` from the arch-chroot.
7. **Now flash the kernel we copied to the root usb earlier**
9. cd /
10. `dd if=repacked2 of=/dev/mmcblk0p2`
11. `cp -R /usr/lib/modules/3.10.18 /mnt/usr/lib/modules/`
12. `umount /mnt`
13. `shutdown -h now`
14. After shutting down you will see a kernel panic, press and hold power to turn off. **required to do this on every shutdown, hopefully someone can fix!**
