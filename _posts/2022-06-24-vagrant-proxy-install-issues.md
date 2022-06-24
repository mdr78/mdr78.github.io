---
layout: default
title: Fixing vagrant-proxy installation problems!
date: 2022-06-24 00:00:00 +0000
tags: Vagrant
---

Always been a big fan of [Vagrant](https://www.vagrantup.com/), [LibVirt](https://github.com/vagrant-libvirt) and [Vagrant Proxy](https://github.com/tmatilai/vagrant-proxyconf). These all rock, as  
fast and convent ways to build and deploy virtual machines on [Linux KVM](https://www.linux-kvm.org/page/Main_Page).  
**Vagrant Proxy** is particularly useful for corporate environments that are  
encumbered by a proxy. It auto-magically detects the host's proxy configuration  
and configures the virtual machine in the same way, as the VM is being built and  
deployed.  

Recently I have been getting alot of *misconfigured* errors when trying to  
install Vagrant Proxy on **Fedora 32**, as follows:  

    $ vagrant plugin install vagrant-proxy
    Installing the 'vagrant-proxy' plugin. This can take a few minutes...
    Vagrant failed to properly resolve required dependencies. These
    errors can commonly be caused by misconfigured plugin installations
    or transient network issues. The reported error is:
    
    conflicting dependencies fog-libvirt (>= 0.6.0) and fog-libvirt (= 0.5.0)
      Activated fog-libvirt-0.5.0
      which does not match conflicting dependency (>= 0.6.0)
    
      Conflicting dependency chains:
        fog-libvirt (= 0.5.0), 0.5.0 activated
    
      versus:
        vagrant-libvirt (> 0), 0.1.0 activated, depends on
        fog-libvirt (>= 0.6.0)

Turns out that is a dependency mismatch, because I am using upstream Vagrant  
instead of the version installed by the Fedora package manager. I came across  
the solution, which is to tell the Vagrant plugin installer where to find its  
dependencies, as follows:  

    $ export CONFIGURE_ARGS="with-libvirt-include=/usr/include/libvirt with-libvirt-lib=/usr/lib64"
    $ vagrant plugin install vagrant-proxy
    Installing the 'vagrant-proxy' plugin. This can take a few minutes...
    Building native extensions. This could take a while...
    Fetching vagrant-proxy-0.0.1.gem
    Installed the plugin 'vagrant-proxy (0.0.1)'!

Similar issues with the Vagrant **LibVirt** plugin are resolved in the same way.  