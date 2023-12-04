## How To Chroot into an Ubuntu ZFS-On-Root: Encrypted, Ubuntu Installer

Target systems are Debian Based; Tested on Ubuntu Systems. Sometimes updates of package 'zfs-dkms' fails to build the zfs modules, from a regression issue, where the code checks for supported kernel versions, and it does not recognise that it actually supports newer kernel versions. Or when adding kernels from the Ubuntu mainline repo. 


## Details

If you have a non-booting ZFS Encrypted installation, that was installed via the OpenZFS styled instructions and inside of LUKS encrypted containers, you are on the correct instructions 


## Reporting 

If you have a thread open at UbuntuForum.org, if you come up with an error on any command, stop right there and post back before going on...


## Deciphering The Technical Details
If you just want to dive in withiut understadning the techincal details under the covers, you can skip this section. If you want to understand they why's and how it works, read on.

bpool connot presently be encyted, becaseu Grub2 cannot find things inside of a bpool that is either inside of a LUKS container, nor inside a native ZFS encrytpted container. I may find a way aorund this in the fututre, but this is what is currently possible.


## Step 1: Documentation References

Get Documentation of where your system is now. Write the URL of where this report uploads to: 

    sudo add-apt-repository ppa:mafoelffen/system-info
    
    sudo apt update
    
    sudo apt install system-info --details

That report, started with those options, will add details related to ZFS, that you may need to adjust things in this documetnation to what may be different on your system.


## Step 2: Boot System
Boot from a recent installer LiveUSB... Use "Try". Open a terminal session.

    sudo su -

Ensure cryptsetup is installed in the LiveEnv

    apt install crytpsetup --yes

Look for where rpool may be

    lsblk -e7 -o name,label,size,fstype

# Unlock The LUKS Container
If the output above has a /dev/zd0, with a crypto_LUKS type, it is hybrid and use the Ubuntu Onstaller type instrcctions. 

If you see another traditonal LUKS partiton that looks like the size could hold an rpool, use that as the device name. What is given in the below command, is just an example. Adapt to shat your output says. 

Note that during the unlock, at the end of that command is a mapper deivce name, this could have been anything that the original person who installed named it. To confirm it works right, you first need to intially unlock the luks conatinaer, mount rpool, to read the cryttab entry's uuid, then look at hte Dev Mapper name... I'll noted what to do if that is different than what we use here, at that point. In that case, after locking it back up, you start here again with that Dev Mapper name.

    cryptsetup luksOpen /dev/sda4 luks1

Mount the pools:

    zpool export -a

    zpool import -N -R /mnt rpool  
    
If this errors, and says it was previously used by another system, add "-f" right before "-N"

    zpool import -N -R /mnt bpool  

If this errors, and says it was previously used by another system, add "-f" right before "-N"

Do this to look at the structure:

    lsblk -e7 -o name,label,size,fstype

This is sample output:
> **&#9432;** NAME     LABEL                   SIZE FSTYPE
> **&#9432;** sr0      Ubuntu 22.04 LTS amd64  3.4G iso9660
> **&#9432;** zd0                              500M crypto_LUKS
> **&#9432;** └─zfskey keystore-rpool          484M ext4
> **&#9432;** vda                               30G 
> **&#9432;** ├─vda1                           512M vfat
> **&#9432;** ├─vda2                           1.4G crypto_LUKS
> **&#9432;** ├─vda3   bpool                   1.5G zfs_member
> **&#9432;** └─vda4   rpool                  26.7G zfs_member

Check that is unlicked & mapped, so it can be refrenced:

    ls /dev/mapper
> **&#9432;** control  luks1

Look at rpool

    zfs list

Adapts the mounts of bpool and rpool from the output of 'zfs list'

    zfs mount $(zfs list | grep -m1 -e '^rpool\/ROOT\/ubuntu_' | awk '{print $1}' )
    
    zfs mount $(zfs list | grep -m1 -e '^bpool\/BOOT\/ubuntu_' | awk '{print $1}' )

    zfs mount -a

Check the layout and mounts:

    zfs list

Output should be simar to thism with differernt UID's ad user name.
zfs mount $(zfs list | grep -m1 -e '^rpool\/ROOT\/ubuntu_' | awk '{print $1}' )
> **&#9432;** NAME                                               USED  AVAIL     REFER  MOUNTPOINT
> **&#9432;** bpool                                              270M  1010M       96K  /mnt/boot
> **&#9432;** bpool/BOOT                                         269M  1010M       96K  none
> **&#9432;** bpool/BOOT/ubuntu_5ylu87                           269M  1010M      269M  /mnt/boot
> **&#9432;** rpool                                             9.65G  16.0G      192K  /mnt
> **&#9432;** rpool/ROOT                                        9.00G  16.0G      192K  none
> **&#9432;** rpool/ROOT/ubuntu_5ylu87                          9.00G  16.0G     5.35G  /mnt
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/srv                       192K  16.0G      192K  /mnt/srv
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/usr                       580K  16.0G      192K  /mnt/usr
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/usr/local                 388K  16.0G      388K  /mnt/usr/local
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var                      3.65G  16.0G      192K  /mnt/var
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/games                 192K  16.0G      192K  /mnt/var/games
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/lib                  3.64G  16.0G     3.48G  /mnt/var/lib
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/lib/AccountsService   212K  16.0G      212K  /mnt/var/lib/#AccountsService
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/lib/NetworkManager    260K  16.0G      260K  /mnt/var/lib/NetworkManager
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/lib/apt              98.2M  16.0G     98.2M  /mnt/var/lib/apt
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/lib/dpkg             61.3M  16.0G     61.3M  /mnt/var/lib/dpkg
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/log                  10.4M  16.0G     10.4M  /mnt/var/log
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/mail                  192K  16.0G      192K  /mnt/var/mail
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/snap                 2.64M  16.0G     2.64M  /mnt/var/snap
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/spool                 276K  16.0G      276K  /mnt/var/spool
> **&#9432;** rpool/ROOT/ubuntu_5ylu87/var/www                   192K  16.0G      192K  /mnt/var/www
> **&#9432;** rpool/USERDATA                                     142M  16.0G      192K  /mnt
> **&#9432;** rpool/USERDATA/user_name_hsfoy4                 142M  16.0G      142M  /mnt/home/mikeferreira
> **&#9432;** rpool/USERDATA/root_hsfoy4                         360K  16.0G      360K  /mnt/root
> **&#9432;** rpool/keystore                                     518M  16.5G     63.4M  -

Look at /etc/cryttab at the dev mapper name for rpool. The UUID of the partition, use the device name you used to unlock that partition in this coomand, and match that UUID with what it in the /etc/cryptab

    grep '$(blkid -s UUID -o value /dev/sda4 )' /etc/cryptab 

If it is 'luks1', skip this next step and continue. If not, then lock the luks container back up, go to the LUKS Unlock step and start back from there which the correct LUKS container Dev mapper name for rpool.

   cryptsetup luksClose luks1

Then go back to the LUKS Open Step


## Mount the sysfs

    mount --make-private --rbind /dev  /mnt/dev
    mount --make-private --rbind /proc /mnt/proc
    mount --make-private --rbind /sys  /mnt/sys
    mount --make-private --rbind /run  /mnt/run

## Chroot in

    cd /mnt
    chroot /mnt /bin/bash --login
    mount -a

Do what you need to do...

## Get Good External Backups.

<<Refer & link to "Full ZPool Backup To External Storage" Doc. Does not exist yet. Planned.>>
I would do first, before doing anything, connect and mount to an external drive and rsync all the data you need to backup off it, so you can rstore it if something goes more wrong.


## After You Are Through
After you are through, to exit gracefully

    exit
    mount | grep -v zfs | tac | awk '/\/mnt/ {print $3}' | \
    xargs -i{} umount -lf {}

    zpool export -a

    reboot


## Conclusion / Further
Fortunately, ZFS is farely flexible, and resilient. 
