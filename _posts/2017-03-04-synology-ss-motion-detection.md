---
layout: post
title:  "Offloading motion detection from your Synology Diskstation"
date:   2017-02-20 13:10:44 +0000
categories: Synology
---

I love my Synology DiskStation DS213j, and what's not to love. It's does a great job and has given me years of trouble-free service. It's got a _fantastic_  UI. Synology do a great job of keeping it up to date with regular updates. There is a library of useful apps for it, some of my favourites are Photo Station, Cloud Backup and Surveillance Station. 

I use Surveillance Station to talk to my 3 x D-Link IP Cameras; its detects motions and archives footage. The trouble is that motion detection is a computationally expensive process and the DS213j shipped with a fairly modest 1x ARMv7 Core @ 1.2 GHz. It consumes quiet bit of the DiskStation's limited CPU horse power and make's other apps like Photo Station pretty sluggish. 

![Resource Utilization 1]({{ site.url }}/images/motion_diskstation/DS-resource-utilization1.png)

The DiskStation spawn's a seperate `ssd` process for each IP camera it manages, for my 720p camera's each instance consumes about 10% of the available CPU. 

![Resource Utilization 2]({{ site.url }}/images/motion_diskstation/DS-resource-utilization2.png)

# Minnowboard Max #

As it happens, I had a spare [Minnowboard Max](http://wiki.minnowboard.org/MinnowBoard_MAX) with an Dual Core Intel ATOM E3800 looking for work. The E3800's  SSE 4.2 vector instruction support, makes it a far better choice for image processing. The Minnowboard supports most off-the-shelf Linux Distributions, but being a minimalist I went for [Buildroot](https://buildroot.uclibc.org/) installing only the usual suspects; Busybox, Connman, SystemD, SSHd and (of course) [Motion] (https://motion-project.github.io/). 

If you have a spare a Minnowboard, you can grab my Buildroot [defconfig]({{ site.url }}/other/minnowboard_max_motion_defconfig) for the Minnowboard Max and place it in Buildroot configs directory. `make minnowboard_max_motion_defconfig` to create your `.config`, then `make` and copy the resulting image to an SD Card. If you don't happen to have a spare Minnowboard, you can use another Board with a CPU with sufficent horse power for the task, along with a Linux Distro that packages [Motion](https://github.com/Motion-Project/motion/releases).

# Configuring Motion #

After installation, Motion typically runs as a SystemD service, you can enable the service with `systemctl enable motion`. You will find Motion's configuration file in the `/etc/motion.conf`. If you are only running a single IP camera, you can put all configuration in the single config file. If you are running a number of cameras, you are better off putting global values in `/etc/motion.conf` and then putting camera specific config in `/etc/motion/motion.camera0.conf`, `/etc/motion/motion.camera1.conf` etc. You should restart Motion after any changes with `systemctl restart motion`. 

One of the most useful capabilities of Motion is it's ability to produce debug images and videos, to show you percisely *what* trigger recording. This is a *huge* benefit of this approach, as Synology's inbuilt Motion Detection can often leave you confused as to what motion actually triggered the capture. You can enable this with the following settings.

	############################################################                            
	# FFMPEG related options                                                     
	# Film (movies) file output, and deinterlacing of the video input         
	# The options movie_filename and timelapse_filename are also used                                                                 
	# by the ffmpeg feature                                                                                                    
	############################################################                  
                                                                                    
	# Use ffmpeg to encode movies in realtime (default: off)               
	ffmpeg_output_movies on                                                            
                                                                                    
	# Use ffmpeg to make movies with only the pixels moving                     
	# object (ghost images) (default: off)                                          
	ffmpeg_output_debug_movies off

You can see a sample debug video this produces, the box showing the pixels that are changing frame-to-frame and are triggering motion detection.

<iframe width="560" height="315" src="https://www.youtube.com/embed/mdNXnHY2v50?ecver=1" frameborder="0" allowfullscreen></iframe>

Camera specific settings are defined in `motion.cameraX.conf`, these override the global setting. Typically in here I define the camera's resolution (width/height), it streaming address (rtsp://), credentials (userpass).

	# Image width (pixels). Valid range: Camera dependent, default: 352
	width 1024
	# Image height (pixels). Valid range: Camera dependent, default: 288
	height 768

	# URL to use if you are using a network camera, size will be autodetected (incl http:// ftp:// mjpg:// rtsp:// mjpeg:// or file://
	# Must be a URL that returns single jpeg pictures or a raw mjpeg stream. A trailing slash may be required for some cameras.
	# Default: Not defined

	netcam_url rtsp://a.b.c.d/live1.sdp

	# Username and password for network camera (only if required). Default: not defined
	# Syntax is user:password
	netcam_userpass username:password

The two most interesting settings here are area_detect, this allows you to define a detection area and exclude know area's of noise (trees etc). on_area_detected allows you to run a command when motion is detected within the defined detection area. 

	area_detect 2,5,8,3,6,9

	on_area_detected /usr/bin/camera_on_garage.sh

Finally I override the default movie filename format to include the camera name, so I can more easly distinguish the debug output of each camera.

	movie_filename %v-%Y%m%d%H%M%S-rear

Motion detection is relatively inexpensive on the Minnowboard Max.

![Resource Utilization 2]({{ site.url }}/images/motion_diskstation/DS-minnowboard-max.png)

# Configuring Synology Surveillance Station #

Synology Surveillance Station has a feature called 'action rules', that enables the defintion of per camera events and actions. 

![Resource Utilization 2]({{ site.url }}/images/motion_diskstation/DS-surveillance-station1.png)

With `action rules` we can cause camera recording to be triggerred by a Web API. In my case I create a rule for each of my cameras, that triggers recording for 90 seconds on each motion event. Each event has it's own event id, in the screenshot below I show the rule `External event [3]` with the event id 3.

![Resource Utilization 2]({{ site.url }}/images/motion_diskstation/DS-surveillance-station2.png)

You can then use curl to trigger the event id 3, and this is what I do in the script `/usr/bin/camera_on_garage.sh` called by on_area_detected.

	#cat /usr/bin/camera_on_garage.sh 
	#!/bin/sh
	curl -k "http://diskstation:5000/webapi/entry.cgi?api=SYNO.SurveillanceStation.ExternalEvent&method=\"Trigger\"&version=1&eventId=3&eventName=\"This is external event3\"&account=\"username\"&password=\"password\""# 
	#

# Performance #

The result of all this sitching together is a mucher happier Synology Diskstation, with a 20-30% reduction in over all CPU utilisation. 

![Resource Utilization 4]({{ site.url }}/images/motion_diskstation/DS-resource-utilization4.png)

As you might expect, individual `ssd` instances are consuming less cycles.

![Resource Utilization 4]({{ site.url }}/images/motion_diskstation/DS-resource-utilization4.png)

And finally, I would also add a big improvement in user experience as motion detection event are now much easier to understand with Motion debug output. 
