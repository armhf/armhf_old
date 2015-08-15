---
title: BeagleBone Black GA Angstrom Image
author: John Clark
layout: post
permalink: /beaglebone-black-ga-angstrom-image/
dsq_thread_id:
  - 1252872788
categories:
  - BeagleBone Black
  - eMMC
  - HowTo
---
Before booting the BeagleBone Black for the first time, you may want to snap an image of the unbooted the Angstrom load from NVRAM in its virgin state for future reference.  To do this, we need a microSD card to boot the BeagleBone Black and snap the image.  The BeagleBone Black Ubuntu image available on the <a href="http://www.armhf.com/index.php/download/" target="_blank">downloads page</a> will work nicely for this purpose.  Just cat the image using <a href="http://www.armhf.com/index.php/getting-started-with-ubuntu-img-file/" target="_blank">these instructions</a>, then boot it from microSD by holding down the user button at power-on.  Once booted, `ssh` into the device, `sudo su` to root, and get started.  
&nbsp;  
If you need the original image you can find it located on the <a href="http://www.armhf.com/index.php/download/" target="_blank">downloads page</a>.  
&nbsp;  
The internal NVRAM will be seen as the `/dev/mmcblk1` device.

    # ll /dev/mmcblk*
    brw-rw---- 1 root disk 179,  0 Apr 30 03:24 /dev/mmcblk0
    brw-rw---- 1 root disk 179,  1 Apr 30 03:24 /dev/mmcblk0p1
    brw-rw---- 1 root disk 179,  2 Apr 30 03:24 /dev/mmcblk0p2
    <strong><span style="color: #ff0000;">brw-rw---- 1 root disk 179,  8 Apr 30 03:24 /dev/mmcblk1</span></strong>
    brw-rw---- 1 root disk 179, 16 Apr 30 03:24 /dev/mmcblk1boot0
    brw-rw---- 1 root disk 179, 24 Apr 30 03:24 /dev/mmcblk1boot1
    <span style="color: #ff0000;">brw-rw---- 1 root disk 179,  9 Apr 30 03:24 /dev/mmcblk1p1
    brw-rw---- 1 root disk 179, 10 Apr 30 03:24 /dev/mmcblk1p2
    </span>

Capture the image using the `dd` command.

    # dd bs=1M count=1832 if=/dev/mmcblk1 | xz -cz > ./bbb_angstrom_ga.tar.xz

Now is a good time to go get a cup of coffee because this takes a while.  
In the future if you need to restore the image, just do the reverse:

    # xz -cd ./bbb_angstrom_ga.tar.xz > /dev/mmcblk1

&nbsp;