---
layout: post
title:  "Repairing a dead Tronic Power Cube (KH 3106)"
date:   2018-01-19 13:10:44 +0000
categories: Repair
tags: maker
---

My Dad gave me the 'gift' of a completely dead Tronic Power Cube that was bought at some point from Lidl. It *should* be a useful portable battery for camping and power outages. It has a 12v Cigarette Lighter output, and 3v, 4.5v, 6v and 9v barrel connector outputs on it's front. On it's rear it has a set of 12v Banana plugs. It also has a good deal of surge protection, with both an output fuse on it's front and an input fuse on it's rear. It takes a standard 12v 0.5a input barrel connector to charge.

![Tronic Power Cube KH 3106]({{ site.url }}/images/tronic_power_cube/IMG_20180114_193238.jpg)

Needless to say it was completely dead when I got it, it wouldn't hold a charge - thanks Dad, great gift! The Power Cube has a 12v 6-fm-7 Lead Acid Battery internally which is cleary due a replacement. So it turns out that I am not the only one with this problem, [Tech it Out](https://www.youtube.com/channel/UCqQsdFxGy6fymK8a7rX31NA) published a [pretty comprehensive video](https://youtu.be/C5WsBF6hDOk) on replacing the Power Cube's 12v battery, and that should be it, right?


Wrong, now the fun really starts. So it turns out the cheap Ningbo Shunhong Battery the Power Cube is a non-standard size (shown on the right below).

![Tronic Power Cube KH 3106]({{ site.url }}/images/tronic_power_cube/InkedIMG_20180115_194325_LI.jpg)

Now clearly the folks at Ningbo figured out their mistake a some point, because I found that another Ningbo Shunhong Battery that I pulled from a Maplin Car Jump Starter and Compressor, is in fact the correct size (shown on the left above). However the short battery creates a problem because it means that the gap in the Power Cube where the battery is supposed to fit is a full 10mm too short to accommodate a standard sized 6-fm-7 cell.

If you watch the [Tech it Out video](https://youtu.be/C5WsBF6hDOk) to the end, you will see they solve this problem by cutting the bottom out of the Power Cube. This leaves the 12v 6-fm-7 battery dangling out the bottom, suspended from the Power Cubes internal spade power connectors. I don't need to futher elaborate what a **bad idea** it is to leave the weight of a lead acid battery supported by the power connectors alone. You can only imagine what might happen, when some unfortunate person thoughtlessly picks up a Power Cube repaired in this way.

After several attempts to find a cell to fit the Power Cube failed and also an attempt to resurrect the Power Cube's original battery also failed, I went looking for an alternative that didn't involve cutting the bottom out of the Power Cube. I instead replaced the Power Cube's original 7Ah battery with a 3.2Ah version of the same cell, which was 25mm shorter than the orignal Ningbo battery. 

![Tronic Power Cube KH 3106]({{ site.url }}/images/tronic_power_cube/IMG_20180114_174146.jpg)

In order to make the new battery the same height as the original cell, I cut a couple of pieces of old plastic drain pipe to size and bolted them together. It is important to get the new battery's size right as we don't want the cell moving around internally while it is carried, similary it is also important that we use a non-conductive and non-flamable material to plug the gap. I bolted the plastic drain pipe together and then duct-taped it to the battery.

![Tronic Power Cube KH 3106]({{ site.url }}/images/tronic_power_cube/IMG_20180114_180717.jpg)

The result is the new battery now fits snuggly into the gap in the Power Cube.

![Tronic Power Cube KH 3106]({{ site.url }}/images/tronic_power_cube/IMG_20180114_181611.jpg)

So clearly there a postives and negatives to this approach. On the negative side the Power Cube now holds much less charge with the new battery. On the positive size the Power Cube is back working again, is extremely useful even with the reduced cell, and most importantly is **non-lethal** when compared with other solutions to repairing it. 

