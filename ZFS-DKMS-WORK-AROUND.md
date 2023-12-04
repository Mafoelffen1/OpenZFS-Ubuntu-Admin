## How To Uninstall package 'zfs-dkms' And Survive

Target systems are Debian Based; Tested on Ubuntu Systems. Sometimes updates of package 'zfs-dkms' fails to build the zfs modules, from a regression issue, where the code checks for supported kernel versions, and it does not recognise that it actually supports newer kernel versions. Or when adding kernels from the Ubuntu mainline repo. 

But note that... Canonical started adding the 'zfs' modules in-kernel, so package 'zfs-dkms' is no longer needed after 22.04.x. Package 'zfs-dkms' is no longer a default installed package, nor does it need to be there anymore.

## Details
If package zfs-dkms is installed, it will eventaully fail occasionally.
- When package zfs-dkms fials to build kernel modules, it gets a dpkg error that prevents building the ZFS modules: icp.ko  spl.ko  zavl.ko  zcommon.ko  zfs.ko  zlua.ko  znvpair.ko  zunicode.ko  zzstd.ko... in in the /lib/modules/<kernel_version>/kernel/zfs/ folder, returning an Error code 10.
- When it does that, it further prevents any other system updates.
- The ZFS modules will still build without package 'zfs-dkms', manually with package 'zfsutils-linux', and update-intiramfs.
- IMPORTANT-- Uninstalling 'zfs-dkms' itself, without doing anything else, results in a non-bootable system. So other things need to go along with that, so that that doesn't happen. Or if that does happen (resultig in a non-bootable system), the user needs instructions on how to recover from that.


## Reporting 

If you are Ubuntu, You were affected by this (Upstream) bug: ["zfs-dkms 2.1.5-1ubuntu6~22.04.2- Kernel module failed to build"][1]

Please go to that Bug Report at Launchpad.Net (or the specific Bug Report, in your relavent Distribution's Bug Tracking System) to join as affected. This is important so that the upstream bug can be corrected. These instructions are just a work-around to be able to update euntil the underlying problem can be resolved.


## Decide What You Can Live With.

This work-around involves risk. That risk may involve resulting in a non-booting system. This tutorila will ink to other documents that will help you recover from those, if that happens. This decision to use this tutorial may leverage whether: 
- Whether you really need to use zfs-dkms any more... It is a package that is not longer need in current Ubuntu. At least verified from 22.04.x on.
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

That report, started with those options, will add details related to ZFS, that you may need if something goes wrong. 


## Get Good External Backups.

<<Refer & link to "Full ZPool Backup To External Storage" Doc. Does not exist yet. Planned.>>


## Step 2: Remove package 'zfs-dkms'

Do this first (Go not stop or reboot during this process!!!)

    sudo su -
    
    apt update
    
    apt remove --purge zfs-dkms

Important = If you are still running, use this. If not running, and from a LiveUSB, reset this to the target kernel version. This searches in /boot for any installed kernel, and builds an array, populated with those versions.
    
    KVERSIONS=$(ls /boot/initrd.img-* | sed 's/\/boot\/initrd\.img\-//g')

Reinstall target kernel headers. You need at least the target linux-header-* package to be able to build kernel modules. I would look at all the active kernels, and at least install those Linux header packages for this.

    for KVERSION in ${KVERSIONS[@]};
    do
    apt install --reinstall --yes linux-image-$KVERSION linux-headers-$KVERSION \
    linux-modules-$KVERSION linux-modules-extra-$KVERSION;
    done

This does return messages similar to this during the process:

    /etc/kernel/header_postinst.d/dkms:
    * dkms: running auto installation service for kernel 6.2.0-26-generic
    ...done.

But if you reboot right now, it would be a non-booter... So go on.

    apt install --reinstall --yes zfsutils-linux

Rebuild initramfs images. Remember this command to rebuild all the modules...

    update-initramfs -c -k all

Powerdown...


## Step 3: Reboot and Continue
If will boot to a root mainatance prompt. Do not panic. Press the <Enter> key to continue to the prompt. 

    zpool import -f rpool

    zpool import -f bpool

Continue for any other pools you have. For some reason, I found that it seems to export all pools except rpool. rpool will say it already exists. But as you know, for the 'just-in-cases'...

    zfs mount -a

Poweroff by shutting it down "cold".... Boot it up.


[1]: https://bugs.launchpad.net/ubuntu/+source/zfs-linux/+bug/2044630
