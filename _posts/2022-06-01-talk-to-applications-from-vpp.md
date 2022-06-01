---
layout: default
title: Guide to talking to applications from FD.io VPP
date: 2022-06-01 00:00:00 +0000
tags: fd.io Linux
---

A regular newbie question in VPP-land is how to talk to 3<sup>rd</sup> party applications  
from FD.io VPP. The answer depends of course on **what** you want to talk, that is  
which API or Protocol.  

As is always the case with FD.io VPP, there is more than one solution depending  
on your specific requirements. So here is the non-exhaustive list, along with  
a high-level description of each option:  

# MEMIF

MEMIF (memory interface) is supported as native driver in [VPP](https://s3-docs.fd.io/vpp/22.02/interfacing/libmemif/index.html) and [DPDK](https://doc.dpdk.org/guides/nics/memif.html), and as  
a [separate library](https://git.fd.io/vpp/tree/extras/libmemif) to talk to applications. Memif is a **raw packet** interface  
for punting raw Ethernet or IP frames from FD.io VPP or DPDK directly to an  
application using the [libmemif](https://git.fd.io/vpp/tree/extras/libmemif) library, that is an application developed on  
top of the [libmemif](https://git.fd.io/vpp/tree/extras/libmemif) library,  

It supports operating in both interrupt and poll-mode in FD.io VPP and the  
application. It supports multiple queues for multi-core deployments, and ships  
with a [dockerfile](https://git.fd.io/vpp/tree/extras/libmemif/dockerfile) to bootstrap application development. This interface was  
designed very much with container deployments in mind.  

# TAP v2

[TAP v2](https://git.fd.io/vpp/tree/src/vnet/devices/tap/FEATURE.yaml) (aka FastTap) is supported as native driver in VPP, and hairpins  
packets through the Linux Kernel to talk to applications.  

Hairpin-ing means that VPP talks to the Network on one side and Linux on the  
other, and Linux then talks to application. This means that any Linux  
networking application can be supported in this way; **sockets based  
applications**, **raw packet applications** and so on, at the expense of the  
hairpin through the kernel.  

The special sauce here is in the current hairpin, the v1 version of this  
interface would have used the kernel's venerable [AF<sub>PACKET</sub>](https://man7.org/linux/man-pages/man7/packet.7.html) interface to  
hairpin packet which wasn't known for speed. FD.io VPP's TAP v2 however uses  
the kernel's [vhost<sub>net</sub> interface](https://www.redhat.com/en/blog/introduction-virtio-networking-and-vhost-net) instead, with FD.io VPP implementing a Virtio  
endpoint to talk to the Kernel.  

This is a more optimized interface compared to AF<sub>PACKET</sub>, with a rich  
feature-set borrowed from Virtualization, supporting both poll-mode and  
interrupt-driven modes of operation, multiple queues for multi-core  
deployments and so on.  

# AF_XDP

[AF<sub>XDP</sub>](https://www.kernel.org/doc/html/latest/networking/af_xdp.html) is the new API supported by the Kernel, and is similar to TAP v2 above  
in that it can be used by FD.io VPP to hairpin packets to the Kernel. AF<sub>XDP</sub>  
is very much seen as the successor to the Linux's AF<sub>PACKET</sub> interface.  

Again, this means that any Linux networking application can be supported at  
the expense of the hairpin through the kernel; **sockets based applications**,  
**raw packet applications** and so on.  

This is an interface with a rich feature-set, supporting both poll-mode and  
interrupt mode, multiple queues for multi-core deployments. Although as its  
development is still a work-in-progress features like Jumbo Frame support may  
still be absent  

# LIBVCL

[LibVCL](https://git.fd.io/vpp/tree/src/vcl/) is designed for those use cases that want to support **sockets  
applications** with FD.io VPP, but don't want to incur the expense of the  
hairpin through the Linux Kernel necessitated by the TAP v2 and AF<sub>XDP</sub>  
approaches. Those are applications that <span class="underline">want</span> a sockets interface, but <span class="underline">don't  
want</span> to talk through the kernel.  

[LibVCL](https://git.fd.io/vpp/tree/src/vcl/) has it's own optimized native API [VPPCOM](https://git.fd.io/vpp/tree/src/vcl/vppcom.h) providing a [BSD sockets](https://en.wikipedia.org/wiki/Berkeley_sockets) like  
interface. This requires some application rework to talk to this new API and  
link against LibVCL, but it provides the most optimal sockets-like interface  
and a direct path to talk to FD.io VPP without the hairpin.  

[LibVCL](https://git.fd.io/vpp/tree/src/vcl/) has magic trick though, it can also impersonate the kernel in it's  
ldpreload-mode. This means that some sockets applications developed to talk to  
Linux Kernel, can be transparently redirected to talk directly to FD.io VPP  
instead. It achieves this by inserting itself between the application and the  
Linux Kernel at runtime, and intercepting calls to the BSD sockets API and  
redirecting them to [LibVCL](https://git.fd.io/vpp/tree/src/vcl/). Caveat Emptor your-mileage-may-vary in  
ldpreload-mode but it is definitely worth a try.  
