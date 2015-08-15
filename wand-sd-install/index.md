---
title: Wandboard microSD Card Installation
author: John Clark
layout: page
dsq_thread_id:
  - 
---
  1. Insert the microSD Card into your computer and observe which device it registers as by typing `ls /dev/sd*`. If you are uncertain, remove the microSD Card and the entry should go away. Once you know which device your microSD Card is, follow the instructions below replacing `/dev/sdX` with the name of the microSD Card in your system.
  2. Begin partitioning the microSD Card by typing `fdisk /dev/sdX` <ol style="list-style-type: lower-alpha;">
      <li>
        Initialize a new partition table by selecting <b>o</b>, then verify the partition table is empty by selecting <b>p</b>.
      </li>
      <li>
        Create a new partition by selecting <b>n</b> for &#8216;new&#8217;, then <b>p</b> for &#8216;primary&#8217;, and <b>1</b> to specify the first partition. Press <b>enter</b> to accept the default first sector of 2048 then press <b>enter</b> again to select the default last sector.
      </li>
      <li>
        Commit the changes by selecting <b>w</b> to &#8216;write&#8217; the partition table and exit fdisk.
      </li>
    </ol>

  3. Format the partition as ext4 by typing `mkfs.ext4 /dev/sdX1`
  4. Install u-boot to the microSD Card (select single, dual, or quad core as appropriate) 
        wget http://s3.armhf.com/dist/wand/wand-uboot.tar.xz
        tar xJvf wand-uboot.tar.xz
        dd if=wand-uboot/wand-uboot-[solo|dual|quad].imx of=/dev/sdX bs=512 seek=2
        sync

  5. Install the desired root filesystem to the microSD Card (ubuntu trusty in this example)
  6.     wget http://s3.armhf.com/dist/wand/ubuntu-trusty-14.04-rootfs-3.10.17.1-wand-armhf.com.tar.xz
        mkdir rootfs
        mount /dev/sdX1 rootfs
        tar xJvf ubuntu-trusty-14.04-rootfs-3.10.17.1-wand-armhf.com.tar.xz -C rootfs
        umount rootfs

  7. The microSD Card is now ready to boot. Install the microSD Card in the Wandboard module (not the carrier board) and boot. Note that for ubuntu installations, the login userid is **ubuntu** and the password is **ubuntu**. Likewise for debian installations, the login userid is **debian** and the password is **debian**.

**Tip:** The package cache has been flushed to reduce the size of the images. Run `apt-get update` after boot to update the package cache, then run `apt-get upgrade` to ensure the latest updates are installed.