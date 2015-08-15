---
title: Wandboard
author: John Clark
layout: page
dsq_thread_id:
  - 2012573815
---
<div class='content-column one_half'>
  <p>
    &nbsp;
  </p>
  
  <h4>
    Images Available
  </h4>
  
  <ul>
    <li>
      <a href="#trusty">Ubuntu Trusty 14.10</a>
    </li>
    <li>
      <a href="#precise">Ubuntu Precise 12.04.4 LTS</a>
    </li>
    <li>
      <a href="#wheezy">Debian Wheezy 7.5</a>
    </li>
    <li>
      <a href="http://www.armhf.com/wand-sd-install/" target="_blank">Installation Instructions</a>
    </li>
  </ul>
</div>

  


<div class='content-column one_half'>
  <img class="wp-image-945" alt="Wandboard" src="http://www.armhf.com/wp-content/uploads/2013/12/wand.jpg" width="360" height="301" />
</div></p> 

#### **The Wandboard**

* * *The 

<a href="http://www.wandboard.org/" target="_blank">Wandboard</a> is a Cortex-A9 ARM device sporting a single, dual, or quad core Freescale i.MX6 SoC. The design incorporates support for HDMI, gigabit ethernet, sound, SATA, WiFi, and Bluetooth (depending on the model). The platform is developed on a developed not-for-profit basis and is supported by engineers volunteering their time. With <a href="http://www.em.avnet.com/en-us/design/drc/Pages/Wandboard-org-Ultra-low-cost-Platform-based-on-the-Freescale-i-MX6-Multimedia-Applications-Processor.aspx" target="_blank">actual street prices ~10% lower</a> than the <a href="http://www.wandboard.org/index.php/buy" target="_blank">suggested retail price</a>, the Wandboard offers great value and performance.</p> 

&nbsp;

#### **The Goal**

* * *

<span>The goal of this site is to provide <em>easy to deploy</em> stock armhf embedded images that are very stable. The goal is <em>not</em> to provide the absolute latest kernels that may lack stability. This ease of deployment and stability enables the software engineer to focus on software development without the hassles of building operating systems and fussing with bleeding-edge kernel issues.</span>

<a name="trusty"></a>  
<span> </span>

#### **Ubuntu Trusty 14.04**

* * *

This image uses the <a href="http://cdimage.ubuntu.com/ubuntu-core/releases/14.04/release/" target="_blank">Ubuntu 14.04</a> core filesystem from Ubuntu with the minimal meta package applied. The kernel is compiled from the <a href="https://github.com/wandboard-org/linux" target="_blank">Wandboard fork</a> of the mainline Linux kernel git repository. The result is an easy-to-install and stable Linux image.

**Binary Download**

  * [ubuntu-trusty-14.04-rootfs-3.10.17.1-wand-armhf.com.tar.xz][1] (June 3, 2014)  
    md5: 5ed346ebba924c85bd36209ae8875f33
  * [linux-headers-3.10.17.1-wand-armhf.com.tar.xz][2] (June 3, 2014)  
    md5: 2304a383bd96cf00303b809bf24d4c91
  * [wand-uboot.tar.xz][3] (June 3, 2014)  
    md5: 793bcbd2ed3b97af3c9d1f0170cfa8a9
  * [installation instructions][4]

<a name="precise"></a>  
<span> </span>

#### **Ubuntu Precise 12.04.4 LTS**

* * *

This image uses the <a href="http://cdimage.ubuntu.com/ubuntu-core/releases/12.04/release/" target="_blank">Ubuntu 12.04</a> core filesystem from Ubuntu with the minimal meta package applied. The kernel is compiled from the <a href="https://github.com/wandboard-org/linux" target="_blank">Wandboard fork</a> of the mainline Linux kernel git repository. The result is an easy-to-install and stable Linux image.

**Binary Download**

  * [ubuntu-precise-12.04.4-rootfs-3.10.17.1-wand-armhf.com.tar.xz][5] (June 3, 2014)  
    md5: 2e1171f53fb9f3b6ed5b733ac9e79d4c
  * [linux-headers-3.10.17.1-wand-armhf.com.tar.xz][2] (June 3, 2014)  
    md5: 2304a383bd96cf00303b809bf24d4c91
  * [wand-uboot.tar.xz][3] (June 3, 2014)  
    md5: 793bcbd2ed3b97af3c9d1f0170cfa8a9
  * [installation instructions][4]

<a name="wheezy"></a>  
<span> </span>

#### **Debian Wheezy 7.5**

* * *

This image is based on the Debian Wheezy 7.2 core filesystem from <a href="http://wiki.debian.org/InstallingDebianOn/TI/BeagleBone" target="_blank">Live-Build</a>, with packages equivalent to the Ubuntu minimal meta package. The kernel is compiled from the <a href="https://github.com/wandboard-org/linux" target="_blank">Wandboard fork</a> of the mainline Linux kernel git repository. The result is an easy-to-install and stable Linux image.

**Binary Download**

  * [debian-wheezy-7.5-rootfs-3.10.17.1-wand-armhf.com.tar.xz][6] (June 3, 2014)  
    md5: 10bc1acdb268440f1675eff5e8c44528
  * [linux-headers-3.10.17.1-wand-armhf.com.tar.xz][2] (June 3, 2014)  
    md5: 2304a383bd96cf00303b809bf24d4c91
  * [wand-uboot.tar.xz][3] (June 3, 2014)  
    md5: 793bcbd2ed3b97af3c9d1f0170cfa8a9
  * [installation instructions][4]

&nbsp;

<span> </span>

#### **Resources**

* * *

  * <a href="http://wiki.wandboard.org/index.php/Main_Page" target="_blank">Wandboard Wiki</a>
  * <a href="https://github.com/wandboard-org/linux" target="_blank">Wandboard Github Fork</a>
  * <a href="http://wiki.wandboard.org/index.php/Getting_started_with_Yocto_on_Wandboard" target="_blank">Yocto Embedded</a>
  * <a href="http://archlinuxarm.org/platforms/armv7/freescale/wandboard" target="_blank">Arch Linux</a>

&nbsp;

* * *

<p style="font-size: .75em;">
  All software is provided “AS IS,” and any expressed or implied warranties, including, but not limited to, the implied warranties of merchantability and fitness for a particular or any purpose are hereby expressly disclaimed. Any and all liability for any damage or loss resulting from the failure, defect, malfunction, use, or misuse of any software product offered is hereby expressly disclaimed. If you have any doubt at all about this software or Linix, do not download this software.
</p>

* * *

 [1]: http://s3.armhf.com/dist/wand/ubuntu-trusty-14.04-rootfs-3.10.17.1-wand-armhf.com.tar.xz
 [2]: http://s3.armhf.com/dist/wand/linux-headers-3.10.17.1-wand-armhf.com.tar.xz
 [3]: http://s3.armhf.com/dist/wand/wand-uboot.tar.xz
 [4]: http://www.armhf.com/wand-sd-install/
 [5]: http://s3.armhf.com/dist/wand/ubuntu-precise-12.04.4-rootfs-3.10.17.1-wand-armhf.com.tar.xz
 [6]: http://s3.armhf.com/dist/wand/debian-wheezy-7.5-rootfs-3.10.17.1-wand-armhf.com.tar.xz