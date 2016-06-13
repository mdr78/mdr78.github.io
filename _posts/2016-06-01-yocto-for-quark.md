---
layout: post
title:  "Building Yocto for the Intel Galileo"
date:   2016-06-01 13:10:44 +0000
categories: galileo yocto
---

Last year I upstreamed support for the Galileo Gen 1/2 to the [Yocto project](https://www.yoctoproject.org). So before I go any further detail talking about building a homebrew LTE modem based on the Galileo, I need to first describe the steps building a configurable Linux distro for the Intel Galileo 1/2. 

First off - a quick explaination. Yocto is the umbrella for a number of seperate projects Bitbake, Poky, OpenEmbedded and various BSPs.

* Bitbake is the build tool - the tool that interprets recipes, figures how to build a given package, it's dependences, it's ordering relative to other packages.
* Openembedded - provides the recipes and support scripts required to build Linux. 
* Poky - aggregate's Bitbake and Openembedded, with policy to build a reference Linux distribution. 

Finally before we build, the most recent version of Yocto is 2.1 'korgoth'. You will see me reference krogoth below. 

# Quick tutorial for building Yocto the Intel  Galileo. #

1. Clone Poky and checkout krogoth. 

        # git clone http://git.yoctoproject.org/git/poky
        # git checkout -b krogoth origin/krogoth

2. Add meta-intel - the BSP for Intel Boards and checkout krogoth.

        # git clone http://git.yoctoproject.org/git/meta-intel
        # git checkout -b krogoth origin/korgoth

3. Setup the yocto environment, source the open embedded script. This will automagically change your directory to ```poky/yocto_build```.

        # cd poky 
        # . oe-init-build-env yocto_build/

4. Add the meta-intel BSP as a layer.

        # bitbake-layers add-layer ../../meta-intel
        # bitbake-layers show-layers
        layer                 path                                      priority
        ==========================================================================
        meta                  /build/yocto/openembedded-core/meta       5
        meta-intel            /build/yocto/meta-intel                   5
        meta-oe               /build/yocto/meta-openembedded/meta-oe    6

5. Edit your local.conf, and change your machine type to "intel-quark".


        # vi conf/local.conf

        #
        # Machine Selection
        #
        # You need to select a specific machine to target the build with. There are a selection
        # of emulated machines available which can boot and run in the QEMU emulator:
        #
        #MACHINE ?= "qemuarm"
        #MACHINE ?= "qemuarm64"
        #MACHINE ?= "qemumips"
        #MACHINE ?= "qemumips64"
        #MACHINE ?= "qemuppc"
        #MACHINE ?= "qemux86"
        #MACHINE ?= "qemux86-64"
        #
        # This sets the default machine to be qemux86 if no other machine is selected:
        #MACHINE ??= "qemux86"
        MACHINE ?= "intel-quark"

6. Build Yocto, I usually build Poky's minimal image - core-image-minimal. 

        # bitbake core-image-minimal 

        ... wait around 30 minutes ...

7. Use the ```mkgalileodisk``` wic configuration to create a disk image suitable to burn to an sdcard.

        # wic create mkgalileodisk -e core-image-minimal 
        Checking basic build environment...
        Done.
        
        Creating image(s)...
        
        Info: The new image(s) can be found here:
          images/build/mkgalileodisk-201606131600-mmcblk0.direct

        The following build artifacts were used to create the image(s):
          ROOTFS_DIR:                   /build/yocto/openembedded-core/yocto_build/tmp-glibc/work/intel_quark-oe-linux/core-image-minimal/1.0-r0/rootfs
          BOOTIMG_DIR:                  /build/yocto/openembedded-core/yocto_build/tmp-glibc/work/intel_quark-oe-linux/core-image-minimal/1.0-r0/core-image-minimal-1.0/hddimg
          KERNEL_DIR:                   /build/yocto/openembedded-core/yocto_build/tmp-glibc/deploy/images/intel-quark
          NATIVE_SYSROOT:               /build/yocto/openembedded-core/yocto_build/tmp-glibc/sysroots/x86_64-linux


        The image(s) were created using OE kickstart file:
          /build/yocto/meta-intel/scripts/lib/wic/canned-wks/mkgalileodisk.wks

8. You can now burn the image ```mkgalileodisk-xxxxxxxxxxxx-mmcblk0.direct``` to an SD Card using your tool of choice. I prefer [dd](http://man7.org/linux/man-pages/man1/dd.1.html) on Linux and [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) on Windows
