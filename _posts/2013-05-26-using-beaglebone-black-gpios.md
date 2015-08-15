---
title: Using BeagleBone Black GPIOs
author: John Clark
layout: post
permalink: /using-beaglebone-black-gpios/
dsq_thread_id:
  - 1322627509
categories:
  - BeagleBone Black
  - HowTo
---
To control digital input / outputs for the BeagleBone Black, you can use the facilities exposed by the kernel in the `/sys/class/gpio` directory. Note that the BeagleBone White pinouts are different from the BeagleBone Black. Also note that the GPIOs available on the BBW have changed between revisions, so be certain to get the proper technical reference manual for your actual board revision.

After boot, nothing is exported for use, but we do see the four GPIO controllers (`gpio0`, `gpio1`, `gpio2`, and `gpio3`):

    # ls -al /sys/class/gpio
    total 0
    drwxr-xr-x  2 root root    0 May 26 15:40 .
    drwxr-xr-x 45 root root    0 Jan  1  2000 ..
    --w-------  1 root root 4096 May 26 15:48 export
    lrwxrwxrwx  1 root root    0 May 26 15:45 gpiochip0 -> ../../devices/virtual/gpio/gpiochip0
    lrwxrwxrwx  1 root root    0 May 26 15:45 gpiochip32 -> ../../devices/virtual/gpio/gpiochip32
    lrwxrwxrwx  1 root root    0 May 26 15:45 gpiochip64 -> ../../devices/virtual/gpio/gpiochip64
    lrwxrwxrwx  1 root root    0 May 26 15:45 gpiochip96 -> ../../devices/virtual/gpio/gpiochip96
    --w-------  1 root root 4096 May 26 15:45 unexport
    

Note that each GPIO controller is offset by 32 from the previous (0, 32, 64, 96) &#8212; more on this later.

Not all GPIO pins are available by default. A lot of frustration may come from not knowing this &#8212; sometimes when it appears that all should be working, it does not. The microcontroller needs to be reconfigured to enable some of the pin modes. To understand what may be available, you need to study table 10 on page 70 for the P8 header and table 11 on page 72 for the P9 header in the <a href="https://github.com/CircuitCo/BeagleBone-Black/blob/master/BBB_SRM.pdf?raw=true" target="_blank">BeagleBone Black System Reference Manual</a>. Pay close attention to which header is P8 and which is P9 (page 68) because they feel backwards to me.

In this example, I will export P9 header GPIO pins 12, 14, 15, 16. Referencing page 72 mode 7 column, it shows pin 12 to be `gpio1[28]`, pin 14 to be `gpio1[18]`, pin 15 to be `gpio1[16]`, pin 16 to be `gpio1[19]`. All of these pins are on the `gpio1` controller (`gpiochip32`). To get to the GPIO number that should be exported from the kernel, we must add on 32 to each GPIO (64 for `gpio2`, and 96 for `gpio3`). Therefore `gpio1[28]` = 32 + 28 = 60. I will just do all the math here and we will refer to them by the kernel nomenclature and stop with the `gpio1[x]` business as it gets confusing.

    pin 12 --> gpio60
    pin 14 --> gpio50
    pin 15 --> gpio48
    pin 16 --> gpio51
    

Now that we have a relationship between the P9 header physical pins and what the kernel is calling them, we can begin configuration. To make this example easy to follow, I am just going to use bash scripts, but all of the same principles apply to the file manipulation APIs in your favorite language.

In this example, we will be configuring all four pins to be digital outputs. To do so, we need to copy the gpio number we want to export into the kernel gpiolib `/sys/class/gpio/export` file.

    echo 48 > /sys/class/gpio/export
    echo 50 > /sys/class/gpio/export
    echo 51 > /sys/class/gpio/export
    echo 60 > /sys/class/gpio/export
    

Afterward we can see there are four new subdirectories for each of the gpios:

    # ls -al /sys/class/gpio
    total 0
    drwxr-xr-x 2 root root    0 May 26 15:40 ./
    drwxr-xr-x 45 root root   0 Jan  1  2000 ../
    --w------- 1 root root 4096 May 26 16:25 export
    lrwxrwxrwx 1 root root    0 May 26 16:25 gpio48 -> ../../devices/virtual/gpio/gpio48/
    lrwxrwxrwx 1 root root    0 May 26 16:25 gpio50 -> ../../devices/virtual/gpio/gpio50/
    lrwxrwxrwx 1 root root    0 May 26 16:25 gpio51 -> ../../devices/virtual/gpio/gpio51/
    lrwxrwxrwx 1 root root    0 May 26 16:25 gpio60 -> ../../devices/virtual/gpio/gpio60/
    lrwxrwxrwx 1 root root    0 May 26 15:45 gpiochip0 -> ../../devices/virtual/gpio/gpiochip0/
    lrwxrwxrwx 1 root root    0 May 26 15:45 gpiochip32 -> ../../devices/virtual/gpio/gpiochip32/
    lrwxrwxrwx 1 root root    0 May 26 15:45 gpiochip64 -> ../../devices/virtual/gpio/gpiochip64/
    lrwxrwxrwx 1 root root    0 May 26 15:45 gpiochip96 -> ../../devices/virtual/gpio/gpiochip96/
    --w------- 1 root root 4096 May 26 15:45 unexport
    

And the contents of one of the gpio subdirectories:

    # ls -alH /sys/class/gpio/gpio48
    total 0
    drwxr-xr-x  3 root root    0 May 26 16:25 .
    drwxr-xr-x 10 root root    0 May 26 15:45 ..
    -rw-r--r--  1 root root 4096 May 26 16:29 active_low
    -rw-r--r--  1 root root 4096 May 26 16:29 direction
    -rw-r--r--  1 root root 4096 May 26 16:29 edge
    drwxr-xr-x  2 root root    0 May 26 16:29 power
    lrwxrwxrwx  1 root root    0 May 26 16:25 subsystem -> ../../../../class/gpio
    -rw-r--r--  1 root root 4096 May 26 16:25 uevent
    -rw-r--r--  1 root root 4096 May 26 16:29 value
    

There are several configuration options available to each gpio. I suggest you read the <a href="https://www.kernel.org/doc/Documentation/gpio/gpio.txt" target="_blank">Linux GPIO Interfaces manual</a> for all of the details. I am only covering the basics here, and the <a href="https://www.kernel.org/doc/Documentation/gpio/gpio.txt" target="_blank">Linux GPIO Interfaces manual</a> is a very important read to understanding Linux gpio control.

Next we must specify if the exported gpio is input or output. It does not make sense to configure anything else ahead of this and as far as I know, the kernel doesn&#8217;t let you do anything else with it until you set the gpio as input or output.

To set a gpio as output, we set its `direction` as `in` or `out`. For outputs there is an alternative nomenclature where output direction can be set instead as `high` or `low` to help with glitch free operation. I&#8217;ll use this nomenclature:

    echo high > /sys/class/gpio/gpio48/direction
    echo high > /sys/class/gpio/gpio50/direction
    echo high > /sys/class/gpio/gpio51/direction
    echo high > /sys/class/gpio/gpio60/direction
    

Pins 12, 14, 15, 16 are now configured as outputs and are currently high &#8212; these pins will now be reading 3.3v. <span style="text-decoration: underline;">Do not use 5v TTL logic parts or you will damage the board.</span>

Finally, we will set the pins low which will depower them:

    echo 0 > /sys/class/gpio/gpio48/value
    echo 0 > /sys/class/gpio/gpio50/value
    echo 0 > /sys/class/gpio/gpio51/value
    echo 0 > /sys/class/gpio/gpio60/value
    

Putting this all together, this script will configure and run a binary counter that will overflow and start at zero again. It is kind of cool to watch if you hook relays up to the board.

    #!/bin/bash -e
    
    if [ ! -d /sys/class/gpio/gpio48 ]; then echo 48 > /sys/class/gpio/export; fi
    if [ ! -d /sys/class/gpio/gpio50 ]; then echo 50 > /sys/class/gpio/export; fi
    if [ ! -d /sys/class/gpio/gpio51 ]; then echo 51 > /sys/class/gpio/export; fi
    if [ ! -d /sys/class/gpio/gpio60 ]; then echo 60 > /sys/class/gpio/export; fi
    
    echo low > /sys/class/gpio/gpio48/direction
    echo low > /sys/class/gpio/gpio50/direction
    echo low > /sys/class/gpio/gpio51/direction
    echo low > /sys/class/gpio/gpio60/direction
    
    for (( i=0 ; ; ++i ))
    do
       if (( i > 0x0f )); then
          i=0
          printf '\n[press  + c to stop]\n\n'
       fi
    
       bit0=$(( (i & 0x01) > 0 ))
       bit1=$(( (i & 0x02) > 0 ))
       bit2=$(( (i & 0x04) > 0 ))
       bit3=$(( (i & 0x08) > 0 ))
       echo $bit3 $bit2 $bit1 $bit0
    
       echo $bit0 > /sys/class/gpio/gpio60/value
       echo $bit1 > /sys/class/gpio/gpio50/value
       echo $bit2 > /sys/class/gpio/gpio48/value
       echo $bit3 > /sys/class/gpio/gpio51/value
    
       sleep .2
    done
    

&nbsp;

##### The Script in Action