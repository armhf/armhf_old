---
title: 'Expanding Linux Partitions: Part 1 of 2'
author: John Clark
layout: post
permalink: /expanding-linux-partitions-part-1-of-2/
dsq_thread_id:
  - 1246845418
categories:
  - BeagleBone Black
  - eMMC
  - HowTo
  - Partitioning
---
When installing Linux by unpacking .tar files into empty boot and root file system partitions, you get to decide how the partitions will lay out. When extracting Linux from a raw image file, the partition table is already defined and may not fit your needs and likely does not utilize all space available on the microSD card. For example, the Ubuntu 12.10 .img file available for the BeagleBone Black is sized exactly at 1832MB so it is able to precisely fit into the BeagleBone Black&#8217;s internal eMMC storage.

If you are extracting this image to a microSD card, you will likely have a significant amount of unused space available. With many inexpensive microSD cards sporting capacities of 4GB to 8GB (or more), the afore mentioned 1.8GB partition layout leaves us with some additional work to be done. Fortunately, it is not difficult to take advantage of all available space. I am going to review two different methods for achieving this goal.

The first method may not be immediately obvious, but we can create an additional third partition and mount it into the filesystem at boot time. There are a few compelling reasons for selecting this method. If you are planning to install a graphical desktop you will want to create a swap partition and you should set aside space for this. Additionally, if you are intending to allow external sources to upload information via a web browser or FTP, it is possible to have a volume run out of space. Having a dedicated upload and logging `var` volume will prevent uploads or excessive logging from utilizing more than a predetermined amount of space. Similar benefits can be realized buy mounting the `home` directory this way as well.

The second technique takes advantage of Linux&#8217;s ability to expand a partition in-place on a booted system so long as no data exists beyond the end of that partition. Afterward, the ext2, ext3, or ext4 filesystem can be expanded using `resize2fs` to use this new available space at the end of the volume.

&nbsp;

### Adding a Partition

In this case we are going to dump all extra space into a single `storage` volume where we can mount from `/etc/fstab` as desired. The example is generic enough to be adapted for specific use cases.

&nbsp;

#### Step 1: Start `fdisk`

* * *

    # fdisk /dev/mmcblk0
    
    Command (m for help): p
    
    Disk /dev/mmcblk0: 15.7 GB, 15719727104 bytes
    4 heads, 16 sectors/track, 479728 cylinders, total 30702592 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
            Device Boot      Start         End      Blocks   Id  System
    /dev/mmcblk0p1   *          63       65598       32768    c  W95 FAT32 (LBA)
    /dev/mmcblk0p2           65599     3751935     1843168+  83  Linux
    
    Command (m for help): 
    

In this case notice that the prepackaged 1.8GB image has been expanded onto a 16GB microSD card (notice the `15.7 GB` at the top). To make a new `/dev/mmcblk0p3` parition we type &#8216;n&#8217; for new &#8216;p&#8217; for primary (the default), 3 for the 3rd partition (the default), select the default first sector (3751936 in my case) and the default value for the last sector to use all remaining space. Don&#8217;t worry about mistakes yet, <ctrl> + c or &#8216;q&#8217; for quit will exit the program leaving your microSD unchanged.

    Command (m for help): n
    Partition type:
       p   primary (2 primary, 0 extended, 2 free)
       e   extended
    Select (default p): 
    Using default response p
    Partition number (1-4, default 3): 
    Using default value 3
    First sector (3751936-30702591, default 3751936): 
    Using default value 3751936
    Last sector, +sectors or +size{K,M,G} (3751936-30702591, default 30702591): 
    Using default value 30702591
    
    Command (m for help):
    

If you want to save space for a swap partition, use the +size notation for the end value. For example `+12G` for a 12GB partition. At this point it is wise to review your work before committing changes by using the &#8216;p&#8217; command to print the current configuration choices. When all looks correct, &#8216;w&#8217; will commit your changes to disk and exit.

At this point we should be able to see the new partition as a device.

    # ls -l /dev/mmcblk0*
    brw-rw---- 1 root disk 179,  0 Apr 30 02:21 /dev/mmcblk0
    brw-rw---- 1 root disk 179,  1 Apr 28 23:47 /dev/mmcblk0p1
    brw-rw---- 1 root disk 179,  2 Apr 28 23:47 /dev/mmcblk0p2
    <span style="color: #ff0000;">brw-rw---- 1 root disk 179,  3 Apr 30 02:22 /dev/mmcblk0p3
    </span>

We now have a partition ready for a filesystem.

&nbsp;

#### Step 2: Format the Partition

* * *

To format as an ext4 volume, use `mkfs.ext4`. Likewise, to format as an ext3 volume, use `mkfs.ext3`. Note that the `-L` option sets the volume label and is optional.

    # mkfs.ext4 -L "storage" /dev/mmcblk0p3
    mke2fs 1.42.5 (29-Jul-2012)
    Discarding device blocks: done                            
    Filesystem label=storage
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    843776 inodes, 3368832 blocks
    168441 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=3451912192
    103 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks: 
    	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208
    
    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done   
    

&nbsp;

#### Step 3: Mount the Volume

* * *

The next step is to create a location in the filesystem to mount the new volume. Create a `/mnt/storage` directory for this purpose and mount the volume.

    # mkdir /mnt/storage
    # mount /dev/mmcblk0p3 /mnt/storage
    # df
    Filesystem     1K-blocks   Used Available Use% Mounted on
    /dev/mmcblk0p2   1781432 541188   1148088  33% /
    udev              252780      4    252776   1% /dev
    tmpfs             101648    248    101400   1% /run
    none                5120      0      5120   0% /run/lock
    none              254112      0    254112   0% /run/shm
    none              102400      0    102400   0% /run/user
    /dev/mmcblk0p1     32686   6134     26552  19% /boot/uboot
    <span style="color: #ff0000;">/dev/mmcblk0p3  13132408  32904  12425740   1% /mnt/storage
    </span>

Place an entry in `/etc/fstab` auto-mount the new volume at boot time.

    proc /proc proc defaults 0 0
    /dev/mmcblk0p2      /              auto   errors=remount-ro   0   1
    /dev/mmcblk0p1      /boot/uboot    auto   defaults            0   0
    <span style="color: #ff0000;">/dev/mmcblk0p3      /mnt/storage   auto   defaults            0   0
    </span>

Each time the BeagleBone Black is booted, it will auto-mount the new volume.

&nbsp;

#### Step 4: Additional Mount Points Using Bind Mounts

* * *

Having a heap of storage sitting off at `/mnt/storage` isn&#8217;t the most useful thing in the world. We want to have meaningful subdirectories of the filesystem using this store. For this we use bind mounts. This allows us to move the users&#8217; `/home` directory to the new storage partition. Notice we create a new empty `/home` directory for the one we move.

    # mv /home /mnt/storage
    # mkdir /home
    

Add the bind mount to `/etc/fstab`.

    proc /proc proc defaults 0 0
    /dev/mmcblk0p2      /              auto   errors=remount-ro   0   1
    /dev/mmcblk0p1      /boot/uboot    auto   defaults            0   0
    /dev/mmcblk0p3      /mnt/storage   auto   defaults            0   0
    <span style="color: #ff0000;">/mnt/storage/home   /home          none   defaults,<strong>bind</strong>       0   0
    </span>

To see the bind mounts specify the `-a` option for `df`.

    # df -a
    Filesystem        1K-blocks   Used Available Use% Mounted on
    /dev/mmcblk0p2      1781432 541108   1148168  33% /
    proc                      0      0         0    - /proc
    sysfs                     0      0         0    - /sys
    none                      0      0         0    - /sys/fs/fuse/connections
    none                      0      0         0    - /sys/kernel/debug
    none                      0      0         0    - /sys/kernel/security
    udev                 252780      4    252776   1% /dev
    devpts                    0      0         0    - /dev/pts
    tmpfs                101648    220    101428   1% /run
    none                   5120      0      5120   0% /run/lock
    none                 254112      0    254112   0% /run/shm
    none                 102400      0    102400   0% /run/user
    /dev/mmcblk0p1        32686   6134     26552  19% /boot/uboot
    /dev/mmcblk0p3      1969936   3032   1865172   1% /mnt/storage
    <span style="color: #ff0000;">/mnt/storage/home   1969936   3032   1865172   1% /home
    </span>

This can be done for other directories such as `/tmp` and `/var` as well.

In <a href="http://www.armhf.com/index.php/expanding-linux-partitions-part-2-of-2/" target="_blank">Part 2</a> of this article, I show how to expand the existing `/dev/mmcblk0p2` volume in-place to achieve similar results.