---
layout: post
title:  "Securing your VPS"
date:   2017-02-20 13:10:44 +0000
categories: VPS
tags: networking
---

I recently setup a VPS with [time4vps](https://www.time4vps.eu/) - the reason why I need a VPS, is a subject for a future post. After a couple of days I logged on and was alarmed to find the following.

	$ ssh root@myvps
	Last failed login: Mon Feb 20 16:27:35 EET 2017 from a.b.c.d on ssh:notty
	There were 10012 failed login attempts since the last successful login.
	Last login: Mon Feb 20 16:27:25 2017 from a.b.c.d

A deeper look in `/var/log/secure` is revealing.

	Feb 19 01:31:31 33501 sshd[10970]: Failed password for root from a.b.c.d port 64366 ssh2
	Feb 19 01:31:33 33501 sshd[10970]: Failed password for root from a.b.c.d port 64366 ssh2
	Feb 19 01:31:35 33501 sshd[10970]: Failed password for root from a.b.c.d port 64366 ssh2
	Feb 19 01:31:39 33501 sshd[10970]: Failed password for root from a.b.c.d port 64366 ssh2
	Feb 19 01:31:39 33501 sshd[10970]: error: maximum authentication attempts exceeded for root from a.b.c.d port 64366 ssh2 [preauth]
	Feb 19 01:31:39 33501 sshd[10970]: Disconnecting: Too many authentication failures [preauth]
	Feb 19 01:31:39 33501 sshd[10970]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=a.b.c.d  user=root
	...

Incidently, I geolocated the IP address of the would be hacker to Lianyungang, China. An easy way to stop this kind of brute force password attack, is to disable passwords and use public key authentication instead, so this is what I did. These instructions are good for Fedora 23 only.

	[root@abcd  ~]# cat /etc/redhat-release 
	Fedora release 23 (Twenty Three)

First off setup public key authenication for your user of choice and confirm it is working with `ssh -v`. The test is that you shouldn't be asked for a password to ssh. 

	$ ssh -v root@a.b.c.d
	OpenSSH_7.4p1, OpenSSL 1.0.2k  26 Jan 2017
	...
	debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic
	debug1: Next authentication method: publickey
	debug1: Offering RSA public key: /home/<user>/.ssh/id_rsa
	debug1: Server accepts key: pkalg rsa-sha2-512 blen 535
	debug1: Authentication succeeded (publickey).
	Authenticated to a.b.c.d (via proxy).

Now disable Password authenication by opening `/etc/ssh/sshd_config` in your editor of choice, and adding the following lines. 

	PubkeyAuthentication yes
	ChallengeResponseAuthentication no
	PasswordAuthentication no

Ensure that there are no duplicates of these settings, that they haven't been overriden elsewhere in the `sshd_config`. Now restart the SSH Daemon and your done. 

	systemctl restart sshd
