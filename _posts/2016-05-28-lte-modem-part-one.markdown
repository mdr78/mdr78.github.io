---
layout: post
title:  "Building a homebrew LTE Router Part 1"
date:   2016-05-28 13:10:44 +0000
categories: LTE router
---

Living in the country-side close to the small city (Limerick/Ireland) means that my access to high speed broadband is pretty limited. I do have a 'mom and pop shop' fixed wireless provider that does for netflix and spotify, but with erratic pings and a highly contended network, it was proving to be unsuitable for work. VOIP, Hangouts, Screensharing were all becoming problematic. I needed an alternative solution. 

My house looks down on the city, and I can see a number of Cellular Base Stations from my front porch. My LTE reception is pretty good for all the cellular providers, low pings and much better bandwidth than the fixed wireless. The problem with the cellular providers is that they are prohibitively expensive for data hungry applications like 'netflix and spotify'. So I decided to keep my fixed wireless in place and build an LTE router exclusively for work, with a cheap pay-as-you-go contract. 

First off, [dovado](https://http://www.dovado.com/) do a nice range of LTE routers - so if you don't feel inclined to build one described in this post, you can get one there. I have used these in the past, they work every well and have a nice few tricks - keepalive etc. The only catch with these category of LTE routers is that they are built for USB LTE modems, typically these don't expose connections for external antenna's. Routers from other manufactures usually also have the same problem. I wanted to build something with an external antenna to maximise my signal.

This design uses a first generation Intel Galileo's and I paired it with a mPCIe LTE modem from Huawei, to great effect. I modified a Galileo Layer cake case from Thingiverse, adding two holes for the antenna. 

Bill of materials
 * Intel Galileo Generation 1 (ebay)
 * [Hauwei ME906e M2M LTE Modem](http://www.aliexpress.com/item/ME906E-HUAWEI-4G-3G-100-NEW-Original-Genuine-Distributor-Mini-PCIE-FDD-LTE-4G-WCDMA-GSM/32519394853.html) - €40
 * [M2 to mPCIe Convertor](http://www.ebay.ie/itm/121742008692?_trksid=p2057872.m2749.l2649&ssPageName=STRK%3AMEBIDX%3AIT) - €9
 * 2 x [6" RF pigtail MHF4 to SMA Female](http://www.aliexpress.com/item/RF-pigtail-jumper-cable-6in-6-IPX-IPEX-I-PEX-U-FL-MHF-4-to-SMA/32357824395.html) - €4
 * 2 x [5db LTE Antenna] (http://www.aliexpress.com/item/B593-antenna-2PCS-4G-antenna-Huawei-4G-LTE-router-external-antenna-for-B593-SMA-connector/32246900860.html) - €8
 * [Thermal Glue](http://www.ebay.ie/itm/161959963660?_trksid=p2057872.m2749.l2649&var=460917259168&ssPageName=STRK%3AMEBIDX%3AIT) - €1
 * [Small Heatsink](http://www.aliexpress.com/item/10pcs-lot-Heatsink-14x14x6mm-Aluminum-Cooling-Fin-Radiator-IC-Chp-CPU-GPU-Electronic-Computer-Heat-Dissipation/32555884415.html) - €.5  
 * [Layer Cake case with antenna holes](http://www.thingiverse.com/thing:1595047)
  * 1/2 sheet white acrylic - €6
  * 1/2 hour of cutting time on a Lazersaur - €9
 * 4 x 3.5mm nuts and bolts - ?
 * A cellular simcard, from your provider of choice.  

Total (exluding the Galileo) : €81.50

Some notes on assembling:-
 * Becareful to remove the metal hooks from the side of the ethernet port, otherwise the layer ontop of the Galileo won't fit.
 * I added a heatsink and thermal glue, as the Galileo had a tendancy to run a little hot. When you are cutting the case with the lazer cutter, I suggest make a number of vendilation holes.

Here is the finished article.
 # The complete LTE router #

![The completely LTE router]({{ site.url }}/_images/IMG_20160528_211526.jpg)

In the next post, I will cover the software setup and the benchmarks!
