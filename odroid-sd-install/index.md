---
title: ODROID-XU microSD Card Installation
author: John Clark
layout: page
dsq_thread_id:
  - 
---
  1. Insert the microSD Card into your computer and observe which device it registers as by typing `ls /dev/sd*`. If you are uncertain, remove the microSD Card and the entry should go away. Once you know which device your microSD Card is, follow the instructions below replacing `/dev/sdX` with the name of the microSD Card in your system. 
      * Begin partitioning the microSD Card by typing `fdisk /dev/sdX` <ol style="list-style-type: lower-alpha;">
          <li>
            Initialize a new partition table by selecting <b>o</b>, then verify the partition table is empty by selecting <b>p</b>.
          </li>
          <li>
            Create a boot partition by selecting <b>n</b> for &#8216;new&#8217;, then <b>p</b> for &#8216;primary&#8217;, and <b>1</b> to specify the first partition. Press <b>enter</b> to accept the default first sector and specify <b>+16M</b> for the last sector.
          </li>
          <li>
            Change the partition type to FAT16 by selecting <b>t</b> for &#8216;type&#8217; and <b>e</b> for &#8216;W95 FAT16 (LBA)&#8217;.
          </li>
          <li>
            Next, create the data partition for the root filesystem by selecting <b>n</b> for &#8216;new&#8217;, then <b>p</b> for &#8216;primary&#8217;, and <b>2</b> to specify the second partition. Accept the default values for the first and last sectors by pressing <b>enter</b> twice.
          </li>
          <li>
            Press <b>p</b> to &#8216;print&#8217; the partition table. It should look similar to the one below.
          </li>
          <pre style="padding-left:1em; width:85%; background: #000000; color: #00ff00;"><code style="font-size: 12px; background: #000000; color: #00ff00;">Disk /dev/sda: 15.9 GB, 15931539456 bytes
64 heads, 32 sectors/track, 15193 cylinders, total 31116288 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x811634cb

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1            2048       34815       16384    e  W95 FAT16 (LBA)
/dev/sda2           34816    31116287    15540736   83  Linux</code></pre>
          
          <li>
            Finally, commit the changes by selecting <b>w</b> to &#8216;write&#8217; the partition table and exit fdisk.
          </li>
        </ol>
    
      * Format the Partitions <ol style="list-style-type: lower-alpha;">
          <li>
            Format partition 1 as FAT by typing <code>mkfs.vfat /dev/sdX1</code>
          </li>
          <li>
            Format partition 2 as ext4 by typing <code>mkfs.ext4 /dev/sdX2</code>
          </li>
        </ol>
    
      * Install u-boot to the microSD Card
        wget http://s3.armhf.com/dist/odroid/odroidxu-uboot.img
        dd if=odroidxu-uboot.img of=/dev/sdX bs=512 seek=1
    
      * Install the desired root filesystem to the microSD Card (ubuntu trusty in this example)
        wget http://s3.armhf.com/dist/odroid/ubuntu-trusty-14.04-rootfs-3.4.91.1-odroidxu-armhf.com.tar.xz
        mkdir boot
        mkdir rootfs
        mount /dev/sdX1 boot
        mount /dev/sdX2 rootfs
        tar xJvf ubuntu-trusty-14.04-rootfs-3.4.91.1-odroidxu-armhf.com.tar.xz -C rootfs
        cp rootfs/boot/boot.ini boot
        cp rootfs/boot/zImage boot
        umount boot
        umount rootfs
    
      * The microSD Card is now ready to boot. Note that for ubuntu installations, the login userid is **ubuntu** and the password is **ubuntu**. Likewise for debian installations, the login userid is **debian** and the password is **debian**.</ol> 
    **Tip:** The package cache has been flushed to reduce the size of the images. Run `apt-get update` after boot to update the package cache, then run `apt-get upgrade` to ensure the latest updates are installed.