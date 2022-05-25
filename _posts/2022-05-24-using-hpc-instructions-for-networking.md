---
layout: default
title: Using HPC instructions to accelerate DPDK and FD.io
date: 2022-05-24 00:00:00 +0000
tags: DPDK fd.io AVX-512
---

In 2020 I wrote a series of white-papers describing using the Intel AVX-512 SIMD  
instruction-set (AVX-512) to accelerate packet processing applications. AVX-512  
is well known for its ability to accelerate AES cryptography with AVX-512 Vector  
AES (vAES) instructions. However, what will be counter intuitive to some, is to  
use an instruction-set like AVX-512, that was primarily designed for HPC type  
workloads to accelerate networking.  

When looking at the kinds of optimizations we were using in [DPDK](https:/www.dpdk.org) and [FD.io](https://www.fd.io/) I  
pulled out a number of common threads, and thought to describe them in a series  
of papers. In broad strokes, the non-exhaustive list of the kinds of  
optimizations that are described are:  

-   **Data-structure translation**, that is using shuffle and permute instructions  
    to translate from one data structure to another. A good example of this, are  
    the DPDK i40e and ICE PMDs which do this to translate NIC descriptors into  
    DPDK mbufs. This also involves a bit of bitfield translation also, along the  
    way. You can find these examples in the DPDK source code in the AVX-512  
    variants of DPDK [i40e](https://git.dpdk.org/dpdk/tree/drivers/net/i40e/i40e_rxtx_vec_avx512.c) and [ICE](https://git.dpdk.org/dpdk/tree/drivers/net/ice/ice_rxtx_vec_avx512.c) Poll-Mode-Driver's RX functions.

-   **Parallel-izing table lookups**, the DPDK [FIB](https://git.dpdk.org/dpdk/tree/lib/fib/) and [ACLs](https://git.dpdk.org/dpdk/tree/lib/acl/) libraries both use a  
    similar set of optimizations to accelerate lookups. This involves  
    -   The batching of operations, both libraries support a batch interface where  
        multiple lookups are passed in a single call to the library. e.g.  
        rte\_fib\_lookup\_bulk and rte\_acl\_classify.
    -   The parallel-izing of table offset calculations, involves SIMD adds, shifts  
        etc, e.g. \_mm512\_add\_\* and \_mm512\_srli\_\*, in order to calculate the offset  
        for multiple table entries in parallel.
    -   The parallel-izing of table entry loads, involves SIMD gather operations,  
        e.g. \_mm512\_\*gather\_\* and friends, in order to load multiple tables entries  
        in parallel.

-   **Match and copy operations on large buffers**, these operations all use the  
    wider AVX-512  ZMM registers to operate on 64-byte (512bits) buffers in one  
    operation. Some examples are  
    -   [DPDK](https://git.dpdk.org/dpdk/tree/lib/eal/x86/include/rte_memcpy.h) and [FD.io](https://git.fd.io/vpp/tree/src/vppinfra/memcpy_x86_64.h) both use the wider AVX-512 registers to accelerate memory  
        copy operations, e.g. rte\_memcpy (DPDK), clib\_memcpy (FD.io) and friends.
    -   [FD.io](https://git.fd.io/vpp/tree/src/vnet/classify/vnet_classify.h) classifiers involve a mask and hash operation to calculate the hash of  
        buffer, and a mask and match operation to match a buffer to classifer entry.  
        These both have AVX-512 variants with SIMD AND'ing large buffers with a mask  
        and then SIMD XOR'ing the result.

-   **Caching table lookups**, [FD.io VPP](https://git.fd.io/vpp/tree/src/vnet/ip/vtep.h) Virtual Tunnel Endpoint (vTEP) uses  
    AVX-512 to lookup VTEP IDs, e.g. VXLAN Tunnel IDs, in a simple cache. This  
    enable checks for valid VTEP IDs to be accelerated. This involves  
    -   Splat'ing the VTEP ID for a given packet across the 64bit eight lanes in a  
        AVX-512 register.
    -   The splat'ed value can then be used to match against a cache of eight valid  
        VTEP IDs in a single operation.
    -   If any of the VTEP ID matches, we will get a non-zero result and processing  
        on the packet can continue.

All of these optimizations and more are detailed in the white-papers themselves.  

-   There is a [Solution Brief](https://builders.intel.com/docs/networkbuilders/intel-avx-512-packet-processing-with-intel-avx-512-instruction-set-solution-brief-1617440193.pdf), an executive summary for the manager-types who  
    just want the broad-strokes.
-   There is an [Instruction-set Overview](https://networkbuilders.intel.com/solutionslibrary/intel-avx-512-instruction-set-for-packet-processing-technology-guide), which provides a basic primer on the  
    AVX-512 instruction-set, as well detailing some of the micro-architectural  
    features associated with the instruction-set, including core power  
    utilization and associated uop execution ports.
-   There is a technology guide to [writing packet processing software](https://networkbuilders.intel.com/solutionslibrary/intel-avx-512-writing-packet-processing-software-with-intel-avx-512-instruction-set-technology-guide), which  
    provides a basic primer on writing packet processing software with the  
    AVX-512 instruction-set.
