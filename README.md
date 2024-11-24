# Dual Boot Issues
Regarding problems encountered while updating Windows 11 or Arch Linux. This next section will be about Windows 11 update though.

If it is regarding Arch Linux updating, i.e. `failed to mount /boot`, the section is at the bottom.

This is only for the GRUB bootloader though. Any other bootloader like systemd boot shouldn't have a problem like this.

I have some of the Error Messages that you may have seen when faced with this issue at the bottom of this page.

I'm just a little stubborn since I'm already used to Grub's interface. 

This issue might even be exclusive to dual booting windows and linux with a grub bootloader. After all, I'm still learning.

# Guide to fixing the Grub Bootloader issue after a Windows Update

The problem in this case, is the grub bootloader doesn't come up, or you would see a `grub` or `grubrescue` screen.

This is possibly due to the lack of the vmlinuz package file.

So this one is mainly to reinstall the bootloader via booting into a live USB iso and chrooting into your system. 
Since this method is most robust compared to other methods, from my experience at least.

This problem took me 4 hours to figure out, so this is here to help.

Dual Boot tends to have two EFI partitions, so make sure you're picking the Linux EFI Partition.
(I almost made the mistake which is why I'm mentioning it here.)

## 1. Boot into your Live USB image.

I used Ubuntu 24.04. Just use `Try or Install Ubuntu`. Do not install, of course.

Ubuntu (safe graphics) also works, in case you're using an Nvidia GPU.

(You CAN type the `nvidia-drm.modeset=1`, `nouveau.modeset=0`, `nomodeset` stuff so that your gpu can boot the live iso, but I just prefer the (safe graphics) method now.)

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
error: unknown filesystem.
Entering rescue mode...
grub rescue>
```

```shell
EFI variables are not supported on this system.
EFI variables are not supported on this system.
grub-install: error: efibootmgr failed to register the boot entry: No such file or directory.
```

#Arch Linux after `sudo pacman -Syu`

If for some reason the error screen shows up when you choose Arch in the Grub Bootloader:
```shell
[FAILED] Failed to mount /boot.
[DEPEND] Dependency failed for Local File Systems.
You are in emergency mode. After logging in, type "journalctl -xb" to view
system logs, "systemctl boot" to reboot, or "exit"
to continue bootup.
Give root password for maintenance.
(or press Control-D to continue):
```

`Ctrl D` does nothing. (at least, for me.)
You have a few options. You can basically do the stuff above, or you can enter as root, and reinstall the necessary packages that way.
Just:
`sudo pacman -S linux`

Once you reboot, the problem should go away.

If not, you can follow the steps in the above sections for a surefire way to solve the problem.

The Boot into Live Environment, Chroot into System, Reinstall Grub Bootloader (that is, if you encountered that problem) and other necessary packages method.

