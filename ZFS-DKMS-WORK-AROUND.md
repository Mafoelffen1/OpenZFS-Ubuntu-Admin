## ZFS Uninstall package 'zfs-dkms' And Survive

Target systems are Debian Based; Tested on Ubuntu Systems. Sometimes updates of package 'zfs-dkms' fail to build the zfs modules, from a regression issue, where the code checks for supported kernel versions, and does not recognise that it support newer kernels.
## Details

- When this happens, it gets a dpkg error that prevents building the ZFS modules: icp.ko  spl.ko  zavl.ko  zcommon.ko  zfs.ko  zlua.ko  znvpair.ko  zunicode.ko  zzstd.ko... in in the /lib/modules/<kernel_version>/kernel/zfs/ folder, returning an Error code 10.
- When it does that, it further prevents any other system updates.
- The ZFS modules will still build without package 'zfs-dkms', manually with package 'zfsutils-linux'.
- Uninstalling 'zfs-dkms' itself, without doing anything else, results in a non-bootable system. So other things need to go along with that, so that that doesn't happen. Or if that does happen, the user needs instructions on how to recover from that.


## Reporting 

If you are Ubuntu, You were affected by this (Upstream) bug: ["zfs-dkms 2.1.5-1ubuntu6~22.04.2- Kernel module failed to build"][1]

Please go to that Bug Report at Launchpad.Net (or the specific Bug Report, in your relavent Distribution's Bug Tracking System) to join as affected. This is important so that the upstream bug can be corrected. These instructions are just a work-around to be able to update euntil the underlying problem can be resolved.


## Decide What You Can Live With.

This work-around involved risk. That risk may involve resulting in a non-booting system. This tutorila will ink to other documents that will help you recover from those, if that happens. This decision to use this tutorial may leverage whether: 
- Whether you are ZFS-On-Root. If not, then there is less risk of being non-bootable on reboot.
- Whether ou can live with not applying updates for awhile 
- Whether you can follow technical instructions
- How comfortable you are at a command prompt
- Whether you are in a production environment where security updates and patch need to be applied in a timely manner


## If You Decide To Go On...
You are aware of the risk and accpet you are resposnisbile for waht happens.


## Step 1: Documentation References

Get Documentation of where your system is now. Write the URL of where this report uploads to: 

    sudo add-apt-repository ppa:mafoelffen/system-info
    
    sudo apt update
    
    sudo apt install system-info --details


## Get Good External Backups.

<<Refer & link to "Full ZPool Backup To External Storage" Doc. Does not exist yet. Planned.>>


## Step 2: Remove package 'zfs-dkms'

Do this first (Go not stop or reboot during this process!!!)

    sudo su -
    
    apt update
    
    apt remove zfs-dkms
    
Important = If you are still running, use this. If not running, and from a LiveUSB, reset thsi to the target kernel version
    
    KVERSION=$(uname -r)

reinstall target kernel

    sudo apt install reinstall linux-image-$KVERSION linux-header-$KVERSION linux-module-$KVERSION linux-module-extra-$KVERSION 
    
    apt install reinstall zfsutils-linux

Reinstall Grub2

    apt install --yes \
    grub-efi-amd64 grub-efi-amd64-signed linux-image-generic \
    shim-signed zfs-initramfs
    
    grub-install --target=x86_64-efi --efi-directory=/boot/efi \
    --bootloader-id=ubuntu --recheck --no-floppy

Rebuild initramfs images. Remember this command to rebuild all the modules...

    apt update-intramfs -c -k all

Update Grub2
    
    update-grub


## Step 3: Reboot and Test
If it boots, your are good.

If not refer to one of the "chroot into your installed ZFS System" doc's (planned), depending on how your system is installed:
- Normal installation (Not ZFS-On-Root), with added ZFS zpools.
- ZFS-On-Root- Normal
- ZFS-On-Root- Encrypted. Manually installed in LUKS Containers.
- ZFS-On-Root- Encrypted- Manually installed as native ZFs encrypted pools.
- ZFS-On-Root- Encrypted- Ubuntu Installer systel Encrypted (Uses both LUKS locked encyption "key" to unlock Native encrypted zpool.)

[1]: https://bugs.launchpad.net/ubuntu/+source/zfs-linux/+bug/2044630
