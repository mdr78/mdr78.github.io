---
layout: default
title: Why Fat Binary support in FD.io VPP rocks!
date: 2022-06-10 00:00:00 +0000
tags: fd.io AVX-512
---

It’s worth understanding how VPP Fat Binary (aka Multi-arch variants) support  
works in FD.io VPP, as it is pretty cool.  

FD.io VPP builds some of its source files multiple times, once for each  
supported generation of a given micro-processor. So it will build these source  
files once for [Intel® Skylake](https://en.wikipedia.org/wiki/Skylake_(microarchitecture)), once for [Intel® Ice Lake](https://en.wikipedia.org/wiki/Ice_Lake_(microprocessor)), once for [Intel® Haswell](https://en.wikipedia.org/wiki/Haswell_(microarchitecture))  
and so on. It does this to give the compiler an opportunity to apply  
micro-processor generation specific optimizations during the build, by  
specifying the build parameter `-march=skylake`, `-march=icelake-server` and  
`-march=haswell` respectively. The build system figures out which source files  
need to be built multiple times by looking at those specified in the  
[VNET\_MULTIARCH\_SOURCES](https://git.fd.io/vpp/tree/src/vnet/CMakeLists.txt) list, instead of the [VNET\_SOURCES](https://git.fd.io/vpp/tree/src/vnet/CMakeLists.txt) list which specifies  
those files that are built just once.  

The result of this build system wizardry is that there may be multiple versions  
of any given VPP graph node (called node variants) in the VPP fat binary, one  
for **Skylake**, one for **Ice Lake** … you get the idea by now. When VPP starts-up  
it looks at the micro-processor that it is running on, identifies the generation  
of the micro-processor and selects then optimal variant of a graph node to run.  
Unless you go digging into guts of FD.io VPP, you’ll should be oblivious that  
this variant runtime selection is going on. You can however look at a graph  
node, and then list it’s variants with the `show node <name>` command.  

Note the example below is running on Intel® Cascade Lake, so the `skx` variant  
has the highest priority and the Intel® Ice Lake variant is disabled.  

    vpp# show node ip4-rewrite
    node ip4-rewrite, type internal, state active, index 315
      node function variants:
        Name             Priority  Active  Description
        icl                    -1          Intel Ice Lake
        skx                   100    yes   Intel Skylake (server) / Cascade Lake
        hsw                    50          Intel Haswell
        default                 0          default
    
      next nodes:
        next-index  node-index               Node               Vectors
             0          307                ip4-drop                0
             1          323             ip4-icmp-error             0
             2          250                ip4-frag                0
    
      known previous nodes:
        lookup-ip4-src (81)                lookup-ip4-dst-itf (82)            lookup-ip4-dst (83)
        sr-localsid-un-perf (129)          sr-localsid-un (130)               sr-localsid (131)
        sr-localsid-d (132)                tcp4-output (176)                  ip4-frag (250)
        ip4-punt-redirect (308)            ip4-load-balance (319)             ip4-lookup (320)
        ip4-classify (328)

You can then change the variant that is being used at runtime, with the command  
`set node function`.  

    vpp# set node function ip4-rewrite hsw
    vpp# show node ip4-rewrite
    node ip4-rewrite, type internal, state active, index 315
      node function variants:
        Name             Priority  Active  Description
        icl                    -1          Intel Ice Lake
        skx                   100          Intel Skylake (server) / Cascade Lake
        hsw                    50    yes   Intel Haswell
        default                 0          default
    
      next nodes:
        next-index  node-index               Node               Vectors
             0          307                ip4-drop                0
             1          323             ip4-icmp-error             0
             2          250                ip4-frag                0
    
      known previous nodes:
        lookup-ip4-src (81)                lookup-ip4-dst-itf (82)            lookup-ip4-dst (83)
        sr-localsid-un-perf (129)          sr-localsid-un (130)               sr-localsid (131)
        sr-localsid-d (132)                tcp4-output (176)                  ip4-frag (250)
        ip4-punt-redirect (308)            ip4-load-balance (319)             ip4-lookup (320)
        ip4-classify (328)

Different graph node variants support different micro-processor  
instruction-sets, the **Skylake** variant supports [Intel® AVX2](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions), the **Ice Lake**  
variant supports [Intel® AVX-512](https://en.wikipedia.org/wiki/AVX-512) and the **Haswell** variant supports [Intel®  
SSE-4.2](https://en.wikipedia.org/wiki/SSE4). This gives the compiler the option of using these instruction-sets when  
optimizing the variants. It also gives the Software Engineer the option to  
handcraft micro-processor generation specific optimizations and have them  
incorporated into specific graph node variants. These are wrapped in the MACROS  
`CLIB_HAVE_VEC256`, `CLIB_HAVE_VEC512` and `CLIB_HAVE_VEC128`, which correspond  
to **AVX2**, **AVX512** and **SSE4.2** respectively.  

A good example of this pattern is the function [vlib\_get\_buffers\_with\_offset](https://git.fd.io/vpp/tree/src/vlib/buffer_funcs.h?id=542088597886df774e63f841166721deeffef1c1),  
this function converts buffer indexes into buffer pointers. Note how the  
**AVX-512** variant of [vlib\_get\_buffers\_with\_offset is wrapped in  
\`CLIB\_HAVE\_VEC512\`. The variant](https://git.fd.io/vpp/tree/src/vlib/buffer_funcs.h?id=542088597886df774e63f841166721deeffef1c1) loads 8 buffer indexes at a time, and then  
applies a vector shift, then vector adds the base address to calculate 8 buffer  
addresses in parallel, before doing a parallel store of the 8 buffer addresses.  
There are also **AVX2** and **SSE4.2** versions of this code, which gets called by  
most graph nodes.  

```C
static_always_inline void
vlib_get_buffers_with_offset (vlib_main_t * vm, u32 * bi, void **b, int count,
                       i32 offset)
{
  uword buffer_mem_start = vm->buffer_main->buffer_mem_start;
#ifdef CLIB_HAVE_VEC512
  u64x8 of8 = u64x8_splat (buffer_mem_start + offset);
  u64x4 off = u64x8_extract_lo (of8);
  /* if count is not const, compiler will not unroll while loop
     se we maintain two-in-parallel variant */
  while (count >= 32)
    {
      u64x8 b0 = u64x8_from_u32x8 (u32x8_load_unaligned (bi));
      u64x8 b1 = u64x8_from_u32x8 (u32x8_load_unaligned (bi + 8));
      u64x8 b2 = u64x8_from_u32x8 (u32x8_load_unaligned (bi + 16));
      u64x8 b3 = u64x8_from_u32x8 (u32x8_load_unaligned (bi + 24));
      /* shift and add to get vlib_buffer_t pointer */
      u64x8_store_unaligned ((b0 << CLIB_LOG2_CACHE_LINE_BYTES) + of8, b);
      u64x8_store_unaligned ((b1 << CLIB_LOG2_CACHE_LINE_BYTES) + of8, b + 8);
      u64x8_store_unaligned ((b2 << CLIB_LOG2_CACHE_LINE_BYTES) + of8, b + 16);
      u64x8_store_unaligned ((b3 << CLIB_LOG2_CACHE_LINE_BYTES) + of8, b + 24);
      b += 32;
      bi += 32;
      count -= 32;
    }
…
#endif
```

All this work enables FD.io VPP to automagically optimize for the micro-processor  
generation it is running on, I think it is pretty slick!  
