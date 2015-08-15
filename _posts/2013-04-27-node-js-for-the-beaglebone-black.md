---
title: Node.js for the BeagleBone Black
author: John Clark
layout: post
permalink: /node-js-for-the-beaglebone-black/
dsq_thread_id:
  - 1242966278
categories:
  - BeagleBone Black
  - HowTo
tags:
  - node debian ubuntu
---
<img class="size-full wp-image-298" alt="node.js" src="http://www.armhf.com/wp-content/uploads/2013/04/nodejs-green.png" width="212" height="114" />

To run node.js on the BeagleBone, it needs to be compiled from scratch or you can install a precompiled distribution available on the <a href="/index.php/download/" target="_blank">downloads page</a>. While it seems to be more common to cross-compile for the ARM, I find it easier to natively compile it on the BeagleBone, but it does take a bit longer.

Note that I compile and install as root `sudo su`, so if you get any differing result, keep this in mind.

&nbsp;

#### Step 1: Prerequisites

We will need a compiler to compile the node source. The build requires python for the configuration scripts and gcc for the actual code.

<pre style="padding-left: 30px;"><code># apt-get install python
# apt-get install build-essential
</code></pre>

&nbsp;

#### Step 2: Download Node Source

Download the latest source code from the node.js website. At the time of writing it is version 0.10.5 so adjust this to the desired version. We will unpack it in the current directory. Specify the `-C <path>` option to extract it elsewhere.

<pre style="padding-left: 30px;"><code># wget http://nodejs.org/dist/v0.10.5/node-v0.10.5.tar.gz
# tar xzvf node-v0.10.5.tar.gz</code></pre>

&nbsp;

#### Step 3: Configure

At the time of this writing, there is a problem with the Google V8 Snapshot feature causing node to segmentation fault. Snapshotting helps node start faster and is not a big-deal feature; we will just compile without it.

<pre style="padding-left: 30px;"><code># cd node-v0.10.5
# ./configure --without-snapshot</code></pre>

##### Result:

<pre style="padding-left: 30px;"><code>{ 'target_defaults': { 'cflags': [],
                       'default_configuration': 'Release',
                       'defines': [],
                       'include_dirs': [],
                       'libraries': []},
  'variables': { 'arm_fpu': 'vfpv3',
                 'arm_neon': 0,
                 'armv7': 1,
                 'clang': 0,
                 'gcc_version': 47,
                 'host_arch': 'arm',
                 'node_install_npm': 'true',
                 'node_prefix': '',
                 'node_shared_cares': 'false',
                 'node_shared_http_parser': 'false',
                 'node_shared_libuv': 'false',
                 'node_shared_openssl': 'false',
                 'node_shared_v8': 'false',
                 'node_shared_zlib': 'false',
                 'node_tag': '',
                 'node_unsafe_optimizations': 0,
                 'node_use_dtrace': 'false',
                 'node_use_etw': 'false',
                 'node_use_openssl': 'true',
                 'node_use_perfctr': 'false',
                 'node_use_systemtap': 'false',
                 'python': '/usr/bin/python',
                 'target_arch': 'arm',
                 'v8_enable_gdbjit': 0,
                 'v8_no_strict_aliasing': 1,
                 'v8_use_arm_eabi_hardfloat': 'true',
                 &lt;span style="color: #ff0000;">'v8_use_snapshot': 'false'&lt;/span>}}
creating  ./config.gypi
creating  ./config.mk</code></pre>

&nbsp;

#### Step 4: Compile

We are ready to compile. It is going to take about a half-hour to complete &#8212; go get a cup of coffee.

<pre style="padding-left: 30px;"><code># make</code></pre>

&nbsp;

#### Step 5: Verify

Now that the build has finished, we can verify that all looks well before we install it.

<pre style="padding-left: 30px;"><code># ./node -e 'console.log("het werkt!");'
# ./node -v</code></pre>

&nbsp;

#### Step 6: Install

Now that all looks well, we are ready install it.

<pre style="padding-left: 30px;"><code># make install</code></pre>