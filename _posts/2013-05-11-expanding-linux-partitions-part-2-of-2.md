---
title: 'Expanding Linux Partitions: Part 2 of 2'
author: John Clark
layout: post
permalink: /expanding-linux-partitions-part-2-of-2/
dsq_thread_id:
  - 1282866169
categories:
  - BeagleBone Black
  - eMMC
  - HowTo
  - Partitioning
---
As shown in <a href="http://www.armhf.com/index.php/expanding-linux-partitions-part-1-of-2/" target="_blank">Part 1</a> of this article, extracting Linux from a pre-made .img file can result in unused space at the end of the uSD card. Depending on the size of the uSD card, this could leave a significant amount of space left unused. This article shows how to repartition the uSD card to utilize all of the space on the uSD card. This can be done on an unmounted filesystem or even on a live filesystem. In the example below, the live filesystem of a BeagelBone Black (running the Debian Wheezy image) will be resized to take advantage of the full 4GB of space available on the uSD card used in this example.

#### Step 1: Start `fdisk`

* * *

To get started, list the volumes available on a BeagleBone Black that is booted from a uSD card:

    root@debian-armhf:/# ls -l /dev/mmcblk*
    <span style="color: #ff0000;">brw-rw---T 1 root floppy 179, 0  Jan 1 2000 /dev/mmcblk0
    brw-rw---T 1 root floppy 179, 1  Jan 1 2000 /dev/mmcblk0p1
    brw-rw---T 1 root floppy 179, 2  Jan 1 2000 /dev/mmcblk0p2</span>
    brw-rw---T 1 root floppy 179, 8  Jan 1 2000 /dev/mmcblk1
    brw-rw---T 1 root floppy 179, 16 Jan 1 2000 /dev/mmcblk1boot0
    brw-rw---T 1 root floppy 179, 24 Jan 1 2000 /dev/mmcblk1boot1
    brw-rw---T 1 root floppy 179, 9  Jan 1 2000 /dev/mmcblk1p1
    brw-rw---T 1 root floppy 179, 10 Jan 1 2000 /dev/mmcblk1p2
    

The listing above shows an external uSD card is currently booted as indicated by the `/dev/mmcblk0` entries and the lack of `/dev/mmcblk0boot` entries. When booted from the internal eMMC, the device with the `/dev/mmcblk1boot` entries would instead be at the device zero location as shown here:

    root@debian-armhf:/# ls -l /dev/mmcblk*
    <span style="color: #ff0000;">brw-rw---T 1 root floppy 179, 0  Jan 1 2000 /dev/mmcblk0
    brw-rw---T 1 root floppy 179, 1  Jan 1 2000 /dev/mmcblk0p1
    brw-rw---T 1 root floppy 179, 16 Jan 1 2000 /dev/<strong>mmcblk0boot0</strong>
    brw-rw---T 1 root floppy 179, 24 Jan 1 2000 /dev/<strong>mmcblk0boot1</strong>
    brw-rw---T 1 root floppy 179, 2  Jan 1 2000 /dev/mmcblk0p2</span>
    brw-rw---T 1 root floppy 179, 8  Jan 1 2000 /dev/mmcblk1
    brw-rw---T 1 root floppy 179, 9  Jan 1 2000 /dev/mmcblk1p1
    brw-rw---T 1 root floppy 179, 10 Jan 1 2000 /dev/mmcblk1p2
    

To get started, run `fdisk /dev/mmcblk0` to examine the partitioning of the external uSD card that is currently booted:

    root@debian-armhf:/# fdisk /dev/mmcblk0
    
    Command (m for help): p
    
    Disk /dev/mmcblk0: 3947 MB, <span style="color: #ff0000;">3947888640 bytes</span>
    4 heads, 16 sectors/track, 120480 cylinders, total <span style="color: #ff0000;">7710720 sectors</span>
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): <span style="color: #ff0000;">512 bytes</span> / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x80000000
    
            Device Boot      Start         End      Blocks   Id  System
    /dev/mmcblk0p1   *        2048        4095        1024    1  FAT12
    /dev/mmcblk0p2            4096     3751935     1873920   83  Linux
    
    Command (m for help): 
    

Take a moment to examine what `fdisk` is reporting. The first line indicates that the uSD card is `3947888640 bytes` in size. The 4th line reports the sector size to be `512 bytes`. Some quick math shows that `3947888640 bytes / 512 bytes per sector = 7710720 sectors` as is also reported on the second line. The default &#8220;block&#8221; size is 1024 bytes, but it does not actually report this unit anywhere. The `Start` and `End` columns are reporting 512 byte sectors, and the `Blocks` column is reporting 1024 byte blocks. The way this .img is partitioned makes the relationship easy to see in this case, but it would not be immediately apparent if this were a large hard disk and the numbers were not as magic. The starting block of the partition table at position 2048 reveals that blocks 0-2047 are not being used for user data. This amounts to a full megabyte (`2048 * 512 bytes per block = 1048576 bytes`), and this is the default of `fdisk`. The first partition is 2048 sectors in size, or 1024K, which confirms the block size being 1K.

To calculate the total size of the space used on the disk,  there are 2048 sectors at the head reserved for partitioning, 2048 sectors in mmcblk0p1, and 3,751,936 sectors in mmcblk0p2 for a total of 3,756,032 sectors. With each sector using 512 bytes, this totals 1,923,088,384 bytes. The choice of 1,923,088,384 bytes was used to make this image the exact size of the BeagleBone&#8217;s available eMMC space (note that 1,923,088,384 bytes / 1024^2 = 1834 MB). It is also a good size because not all external 2 GB uSD cards are exactly 2048 * 1024^2 in size. Having it a bit under 2 GB makes it a sure fit.

Repartitioning the disk is rather easy since `fdisk` will prompt with smart default choices. The steps below will begin by deleting partition 2, then recreate it as a larger size, and finally **write the new table to the disk only at the end**. Notice the emphasis on the last part: No changes are actually committed to disk along the way &#8212; only by pressing &#8216;w&#8217; at the end will cause changes to be written. If you make a mistake or panic, just hit &#8216;q&#8217; to quit and no changes will have been made.

&nbsp;

#### Step 2: Delete Partition 2

* * *

Press &#8216;d&#8217; for delete and &#8216;2&#8217; for partition 2.

    Command (m for help): p
    
    Disk /dev/mmcblk0: 3947 MB, 3947888640 bytes
    4 heads, 16 sectors/track, 120480 cylinders, total 7710720 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x80000000
    
            Device Boot      Start         End      Blocks   Id  System
    /dev/mmcblk0p1   *        2048        4095        1024    1  FAT12
    /dev/mmcblk0p2            4096     3751935     1873920   83  Linux
    
    <span style="color: #ff0000;"><strong>Command (m for help): d
    Partition number (1-4): 2</strong></span>
    
    Command (m for help): p
    
    Disk /dev/mmcblk0: 3947 MB, 3947888640 bytes
    4 heads, 16 sectors/track, 120480 cylinders, total 7710720 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x80000000
    
            Device Boot      Start         End      Blocks   Id  System
    /dev/mmcblk0p1   *        2048        4095        1024    1  FAT12
    
    Command (m for help): 
    

&nbsp;

#### Step 3: Recreate Partition 2

* * *

Press &#8216;n&#8217; for new, &#8216;p&#8217; for primary, and &#8216;2&#8217; for partition 2. Specify start and end sectors for the new partition &#8212; just select the default values by pressing enter. In fact, outside of the the first &#8216;n&#8217; they were all default choices and pressing enter alone to confirm the choice is all that is needed.

    <span style="color: #ff0000;"><strong>Command (m for help): n</strong></span>
    Partition type:
       p   primary (1 primary, 0 extended, 3 free)
       e   extended
    <span style="color: #ff0000;"><strong>Select (default p): p</strong>
    <strong>Partition number (1-4, default 2): 2
    First sector (4096-7710719, default 4096): 
    Using default value 4096</strong></span>
    Last sector, +sectors or +size{K,M,G} (4096-7710719, default 7710719): 
    <span style="color: #ff0000;"><strong>Using default value 7710719</strong></span>
    
    Command (m for help): p
    
    Disk /dev/mmcblk0: 3947 MB, 3947888640 bytes
    4 heads, 16 sectors/track, 120480 cylinders, total 7710720 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x80000000
    
            Device Boot      Start         End      Blocks   Id  System
    /dev/mmcblk0p1   *        2048        4095        1024    1  FAT12
    /dev/mmcblk0p2            4096     7710719     3853312   83  Linux
    
    Command (m for help): 
    

That is it!  Select &#8216;w&#8217; to commit the changes to the uSD card.  Notice that the partition table in this example was &#8220;busy&#8221; so a reboot was needed to cause the changes to be reflected.  Even if it were not busy, it seems like it could be a good idea to reboot at this point if you want to be extra safe.

    <span style="color: #ff0000;"><strong>Command (m for help): w</strong></span>
    
    The partition table has been altered!
    
    Calling ioctl() to re-read partition table.
    
    WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
    The kernel still uses the old table. The new table will be used at
    the next reboot or after you run partprobe(8) or kpartx(8)
    Syncing disks.
    root@debian-armhf:/# reboot
    

&nbsp;

#### Step 4: Expand the Filesystem

* * *

This is the last step. Now that the second partition is larger, expand the filesystem to match the larger partition using `resize2fs`.

    
    root@debian-armhf:/# df
    Filesystem     1K-blocks   Used Available Use% Mounted on
    <span style="color: #ff0000;"><strong>rootfs           1811704 740184    977824  44% /</strong></span>
    /dev/root        1811704 740184    977824  44% /
    devtmpfs          253920      0    253920   0% /dev
    tmpfs              50816    216     50600   1% /run
    tmpfs               5120      0      5120   0% /run/lock
    tmpfs             101620      0    101620   0% /run/shm
    /dev/mmcblk0p1      1004    474       530  48% /boot/uboot
    root@debian-armhf:/# 
    
    root@debian-armhf:/# <span style="color: #ff0000;"><strong>resize2fs /dev/mmcblk0p2</strong> </span>
    resize2fs 1.42.5 (29-Jul-2012)
    Filesystem at /dev/mmcblk0p2 is mounted on /; on-line resizing required
    old_desc_blocks = 1, new_desc_blocks = 1
    The filesystem on /dev/mmcblk0p2 is now 963328 blocks long.
    
    root@debian-armhf:/# df
    Filesystem     1K-blocks   Used Available Use% Mounted on
    <span style="color: #ff0000;"><strong>rootfs           3761680 741096   2851404  21% /</strong></span>
    /dev/root        3761680 741096   2851404  21% /
    devtmpfs          253920      0    253920   0% /dev
    tmpfs              50816    216     50600   1% /run
    tmpfs               5120      0      5120   0% /run/lock
    tmpfs             101620      0    101620   0% /run/shm
    /dev/mmcblk0p1      1004    474       530  48% /boot/uboot
    root@debian-armhf:/# 
    

&nbsp;

#### Summary

* * *

All of the space on the uSD card is now available for use. To recap:

    # fdisk /dev/mmcblk0
    d
    2
    n
    p
    2
    4096
    
    w
    
    # resize2fs /dev/mmcblk0p2
    

<a href="http://www.armhf.com/index.php/expanding-linux-partitions-part-1-of-2/" target="_blank">Part 1</a>