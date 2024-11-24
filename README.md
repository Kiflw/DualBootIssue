# Dual Boot Issues
Regarding problems encountered while updating Windows 11 or Arch Linux.

# Guide to fixing the Grub Bootloader issue after a Windows Update

The problem in this case, is the grub bootloader doesn't come up, possibly due to the lack of the vmlinuz package file.
So this one is mainly to reinstall the bootloader.
This problem took me 4 hours to figure out, so this is here to help.

Dual Boot tends to have two EFI partitions, so make sure you're picking the Linux EFI Partition.
(I almost made the mistake which is why I'm mentioning it here.)

## 1. Boot into your Live USB image.

I used Ubuntu 24.04. Just have it at try. Do not install, of course.

## 2. Mounting your Linux filesystem and EFI partition.

```shell
sudo mount /dev/nvme0n1p5 /mnt
sudo mount /dev/nvme0n1p4 /mnt/boot/efi
```

## 3. Chroot into your system

"Change Root", or "chroot" can be used to 'enter' your Linux environment as if you're booted into the system. 
(except that you don't have the GUI.)
```shell
for dir in /dev /proc /sys /run; do sudo mount --bind $dir /mnt$dir; done
sudo chroot /mnt
```
The /dev /proc /sys /run are required here so that you can access those directories in the chroot environment. 
(More detailed explanation at the bottom.)

## 4. Reinstall the Kernel

This is done to reinstall the vmlinuz file. Then you update the grub config file:

If Ubuntu,
```shell
apt update
apt install --reinstall linux-image-generic.
update-grub
```

Arch Linux,
```shell
sudo pacman -Syu linux
sudo pacman -Syu -S efibootmgr
grub-mkconfig -o /boot/grub/grub.cfg
```

after the `grub-mkconfig -o /boot/grub/grub.cfg`, you should see your linux image and windows found by the grub config file.
```shell
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/initramfs-linux.img
Found fallback initrd image(s) in /boot: initramfs-linux-fallback.img
Warning: os prober will be executed to detect other bootable partitions.
Its output will be used to detect bootable binaries on them and create new boot entries.
grub-probe: error: cannot find a GRUB drive for /dev/sdb2. Check your device.map.
Found Windows Boot Manager on /dev/nvme0n1p1@/efi/Microsoft/Boot/bootmgfw.efi
done
```

Don't worry about `/dev/sdb2`. It is my Live USB.

## 5. (Optional, but recommended) Exiting Chroot and Unmount Partitions

```shell
exit
for dir in /dev /proc /sys /run; do sudo umount /mnt$dir; done
sudo umount /mnt/boot/efi
sudo umount /mnt
```


# Why Bind `/dev`, `/proc`, `/sys`, `/run`?

The `/dev` directory is where my nvme drive is. Disks, partitions and peripherals.

`/dev/sda` and `/dev/nvme0n1` are here.

`/proc` contains information about running processes and system resources. 

Commands like `ps`, `top`, `free` read data from `/proc`. 

You might be able to run the bind command without this directory for reinstalling the kernel.

`/sys` is similar to `/proc`, which provides info about the devices and drivers. 

without it, for example, commands like `mkinicpio`, which generates the initramfs, may fail and the system may not detect hardware well.

`/run` contains runtime data for services and processes. 

seems that `grub-install` might look for files in `/run`, if and when you're reinstalling the grub bootloader.

## Error Messages (you may have) encountered

vmlinuz file should be located in the `/boot` directory. If not present, need to reinstall by chroot to regenerate the kernel.

`vmlinuz not found`

```shell
EFI variables are not supported on this system.
EFI variables are not supported on this system.
grub-install: error: efibootmgr failed to register the boot entry: No such file or directory.
```
