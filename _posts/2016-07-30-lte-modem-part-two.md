---
layout: post
title:  "Building a homebrew LTE Router Part 2"
date:   2016-07-30 13:10:44 +0000
categories: LTE router
tags: maker
---
This post is part two of series on building a homebrew LTE router. In this post we dive into the software side of the build. If you haven't see the hardware side of the build described in the [first post](http://mdr78.github.io/lte/router/2016/05/28/lte-modem-part-one.html), its worth a quick scan to give you some context.

The software is packaged as a bitbake layer on top of [Yocto 2.1 Krogoth](https://www.yoctoproject.org/downloads/core/krogoth21). I describe the steps to build Yocto Krogoth for the Intel Galileo Gen 1/2 in a [previous post](http://mdr78.github.io/galileo/yocto/2016/06/01/yocto-for-quark.html), again its worth a quick scan to give you some context.

You can find the layer described in this post in my [meta-lte-modem github repo](https://github.com/mdr78/meta-lte-modem). For the impatent, you can just `git clone` the [meta-lte-modem github repo](https://github.com/mdr78/meta-lte-modem), update the APN, add the  layer to the [krogoth build for Intel Quark](http://mdr78.github.io/galileo/yocto/2016/06/01/yocto-for-quark.html) and then build `core-image-minimal` as normal. As follows ...

	[17:14 ] yocto > git clone https://github.com/mdr78/meta-lte-modem
	Cloning into 'meta-lte-modem'...
	remote: Counting objects: 39, done.
	remote: Compressing objects: 100% (27/27), done.
	remote: Total 39 (delta 3), reused 39 (delta 3), pack-reused 0
	Unpacking objects: 100% (39/39), done.
	Checking connectivity... done.
	
	[17:14 ] echo "APN=MyISPsAPN" > ./meta-lte-modem/recipes-core/initscripts/initscripts/qmi-network.conf
	[17:14 ] cd poky/
	[17:15 ] poky >
	
	17:15 ][silv-debian7-yocto-build 34] poky > . oe-init-build-env yocto_build
	You had no conf/local.conf file. This configuration file has therefore been
	created for you with some default values. You may wish to edit it to, for
	example, select a different MACHINE (target hardware). See conf/local.conf
	for more information as common configuration options are commented.

	...
	You can now run 'bitbake <target>'

	Common targets are:
    		core-image-minimal
    		core-image-sato
    		meta-toolchain
    		meta-ide-support

	[17:16 ] yocto_build > bitbake-layers add-layer ../../meta-lte-modem/
	[17:17 ] yocto_build > bitbake-layers show-layers
	layer                 path                                      priority
	==========================================================================
	meta                  /build/yocto/poky/meta                    5
	meta-poky             /build/yocto/poky/meta-poky               5
	meta-yocto-bsp        /build/yocto/poky/meta-yocto-bsp          5
	meta-lte-modem        /build/yocto/meta-lte-modem               6
	
	[17:12 ][silv-debian7-yocto-build 21] yocto_build > bitbake core-image-minimal
	Parsing recipes:   0% |                                                       | ETA:  00:01:39
	...

When the build is done, generate your sdcard image using `wic` and then `dd` to your as SDcard, all described in the [build instructions](http://mdr78.github.io/galileo/yocto/2016/06/01/yocto-for-quark.html). 

Now diving into the detail of the code. 

# Adding CBC MBIM and QMI support to the kernel #

First you will notice I have tweaked the yocto kernel.

        [21:29 ] meta-lte-modem > find recipes-kernel/
        recipes-kernel/linux/linux-yocto
        recipes-kernel/linux/linux-yocto/natting.cfg
        recipes-kernel/linux/linux-yocto/usb-wlan.cfg
        recipes-kernel/linux/linux-yocto/nomodules.cfg
        recipes-kernel/linux/linux-yocto_4.4.bbappend

If you dig in here, what you will find I am doing is adding kernel configuration fragments to the kernel build. These add kernel support for CDC MBIM, QMI WWAN and NATing to the kernel's configuration. You will also find I have tweaked the packages that are bundled with `core-image-minimal`.

	recipes-core/images
	recipes-core/images/core-image-minimal.bbappend

In the `core-image-minimal` bbappend I add a bunch of the usually suspects dropbear, usbutils etc, however two important packages that are added are libmbim and libqmi, these add support for the CDC MBIM and QMI networking services. 

# Enabling DHCP Server on the Ethernet Port #

I add a `sysctl.conf` to turn on IP forwarding automatically on startup. 

	recipes-core/base-files/base-files/sysctl.conf
	recipes-core/base-files/base-files_3.0.14.bbappend

I changed the interfaces files, so that the Galileo's ethernet port has a static IP address set on it `192.168.128.1`. I also tweak dnsmasq to listen on the Galileo's ethernet port and to allocate IPs in the `192.168.128.0/24` range. Anything that wishes to connect to the internet through the Galileo will get an IP in the `192.168.128.0/24` range and will connect through the gateway `192.168.128.1`. `

	recipes-core/init-ifupdown/init-ifupdown/interfaces
	recipes-core/init-ifupdown/init-ifupdown_1.0.bbappend
	recipes-support/dnsmasq/dnsmasq/dnsmasq.conf
	recipes-support/dnsmasq/dnsmasq_2.75.bbappend

# Connecting to the Cellular network #

I have tweaked the SystemV initscripts to add a service to start the QMI Networking service. The service starts `qmi-network` to do the handshaking with the cellular network, calls `udhcpc` to get an IP address from the cellular provider and add the IP Tables rules for NATing.

	recipes-core/initscripts/initscripts_1.0.bbappend
	recipes-core/initscripts/initscripts
	recipes-core/initscripts/initscripts/wwanservice
	recipes-core/initscripts/initscripts/qmi-network.conf
	recipes-core/initscripts/initscripts/wwan.rules

*The Huawei me906e*

The file `wwan.rules` is a udev rule for Hauwei ME906E. The ME906E is a bit of a chameleon and took some figuring out, thankfully Bjørn Mork the Kernel Mantainer for the "USB QMI WWAN NETWORK DRIVER" was a huge help.  Turns out the ME906E is a bit of a chameleon, it can present itself as three serial devices (yes really!) for a PPP setup, an ethernet interface, a CDC MBIM WWAN interface and finally a QMI WWAN interface - depending on how you configure it.

The ME906E is a USB device, even though it physically connects to the Intel Galileo over mini PCI express. 

	root@intel-quark:~# lsusb
	Bus 001 Device 002: ID 12d1:1573 Huawei Technologies Co., Ltd.
	Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

You can cycle through the various available configurations by writing a value to `bConfigurationValue`.

	root@intel-quark:/sys/devices/pci0000:00/0000:00:14.3/usb1/1-2# cat bConfigurationValue
	1

This is what the `wwan.rules` udev rules does. It tells udev when you see the ME906E device, if `bConfigurationValue` is not currently set to 1, set it to 1. As 1 is the QMI WWAN configuration.  

	root@intel-quark:~# cat /etc/udev/rules.d/wwan.rules
	ATTR{idVendor}=="12d1",ATTR{idProduct}=="1573",ATTR{bConfigurationValue}!="1",ATTR{bConfigurationValue}="1"

Once the value 1 is written to bConfigurationValue, the ME906E disappears and reappears as a QMI network device - ready to talk to the qmi-network service. 

	root@intel-quark:~# dmesg
	...
	qmi_wwan 1-2:1.4: cdc-wdm0: USB WDM device
	qmi_wwan 1-2:1.4 wwan0: register 'qmi_wwan' at usb-0000:00:14.3-2, WWAN/QMI device, 02:bb:23:31:9b:84
	usbcore: registered new interface driver qmi_wwan
	ip_tables: (C) 2000-2006 Netfilter Core Team
	nf_conntrack version 0.5.0 (3618 buckets, 14472 max)
