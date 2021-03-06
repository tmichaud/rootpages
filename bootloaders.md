# Bootloaders
* [GRUB 2](#grub-2)
  * [Configuring GRUB 2](#configuring-grub-2)
  * [Installing GRUB 2](#installing-grub-2)
  * [GRUB 2 Recovery](#grub-2-recovery)

## GRUB 2
GRUB stands for the GRand Unified Bootloader. It was designed to be cross platform compatible with most operating systems including BSD, Linux, and Windows variants.

### Configuring GRUB 2

Important files:

| FILE | DESCRIPTION |
| ---- | ----------- |
| /etc/default/grub | GRUB settings. |
| /etc/grub.d/ | A folder with various scripts that make up the grub.cfg. Scripts prefixed with lower numbers are executed first.
| /boot/grub/grub.cfg | This is automatically generated using the settings from /etc/default/grub and the scripts in /etc/grub.d/. Manual changes may get overwritten. |

Common Options:
* /etc/default/grub
  * GRUB_DEFAULT = The default menu entry to autoboot into. Valid options are:
     * Use the menu position "0", use the selected menu option from the last boot "saved", or verbosely specific the menu name to load.
  * GRUB_TIMEOUT = Set the timeout (in seconds) before booting into the default menu entry.
  * GRUB_CMDLINE_LINUX = Append kernel options to the end of the "linux" line. These can later be seen in the operating system in /proc/cmdline. This applies to both the normal and recovery mode options.
  * GRUB_CMDLINE_LINUX_DEFAULT = The same as the above setting except this option does not affect the recovery kernel options.
  * GRUB_DISABLE_LINUX_UUID = If set to "true", devices from /dev/ will be used for specifying the root instead of the UUID. The default is "false" which will use UUIDs.
  * GRUB_BACKGROUND = Specify the full path to a custom image for GRUB's menu background.
 
[1]
  

Source:
1. "GRUB2/Setup." Ubuntu Documentation. November 29, 2015. https://help.ubuntu.com/community/Grub2/Setup

### Installing GRUB 2

GRUB must be installed onto the start of the entire drive, not a partition, to avoid issues in the case of partitions needing to be modified. The first 512 bytes of a drive are used for the Master Boot Record (MBR). If you are using a GPT partition then it uses the first 2048 bytes. GRUB will add it's own data right after that. It is usually a safe and recommended option to create your first partition 1MB after the start of the drive, especially if GPT is in use.

Install GRUB to a drive (replace "X") and then generate a boot menu configuration file. This will create the menu file that loads up to the end-user upon boot.
```
# grub-install /dev/sdX
# grub-mkconfig -o /boot/grub/grub.cfg
```

If any changes are made to GRUB's settings and/or it's various scripts, run this command to update the changes. [1]
```
# update-grub
```

Common "grub-install" options:
* compress = Compress GRUB-related files. Valid options are:
  * no (default), xz, gz, lzo
* --modules = List kernel modules that are required for boot. Depending on the end-user's setup, "lvm", "raid" (for mdadm), and/or "encrypt" (for LUKS) may be required.
* --force = Install despite any warnings.
* --recheck = Remove the original /boot/grub/device.map file (if it exists) and then review the current mapping of partitions.
* --boot-directory = The directory that the "grub/" folder should exist in. This is typcially "/boot". [2]

Source:

1. "GRUB." Arch Linux Wiki. May 27, 2016. https://wiki.archlinux.org/index.php/GRUB
2. "GRUB2-INSTALL MAN PAGE." Mankier. Feburary 26, 2014. https://www.mankier.com/8/grub2-install

### GRUB 2 Recovery
In cases where GRUB fails (because it was installed incorrectly), the end-user is automatically switched into GRUB's rescue shell.  Only a few commands are available for use.

Common options:
* insmod = Load kernel modules.
* ls = List partitions and file systems within them.
* cat = View file contents.
* set = Set a boot option.
* unset = Remove a boot option.
* boot = Attempt to boot again.
* halt = Shutdown the computer.
* reboot = Restart the computer.

The rescue prompt will look similar to this.
```
grub rescue>
```

Example of using these commands to do a custom rescue boot.
```
grub rescue> ls
(hd0) (hd0,msdos1)
grub rescue> ls (hd0,1)/boot/
grub/
vmlinuz
initramfs-linux.img
grub rescue> set root=(hd0,1)
grub rescue> linux /boot/vmlinuz root=/dev/sda1
grub rescue> initrd /boot/initramfs-linux.img
grub rescue> boot
```

Alternatively, you can switch back to the graphical GRUB menu and make changes there.
```
grub rescue> insmod normal
grub rescue> normal
```

For recovering from a corrupt GRUB installation, fully change root into the environment from a live CD, USB, or PXE network boot. Then you can modify configuration files and re-install GRUB using the same commands used during the installation.

In this example, /dev/sda2 is the root partition and /dev/sda1 is the boot partition.
```
# mount /dev/sda2 /mnt
# mount /dev/sda1 /mnt/boot
# mount --bind /dev /mnt/dev
# mount --bind /run /mnt/run
# mount --bind /sys /mnt/sys
# chroot /mnt
# /bin/bash
# export PATH="$PATH:/sbin:/bin"
```
[1]

If you need to recover GRUB from a chroot that is based on a LVM on the host node, make sure that LVM tools are installed on the guest. This way it can properly see the logical volume as a block device.
```
Debian/Ubuntu
# apt-get install lvm2
```
```
RHEL/Fedora
# yum install lvm2
```

Source:

1. "Grub2/Installing." Ubuntu Documentation. March 6, 2015. https://help.ubuntu.com/community/Grub2/Installing
2. "GNU GRUB Manual 2.00." GNU. Accessed June 27, 2016. https://www.gnu.org/software/grub/manual/grub.html


