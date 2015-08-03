---
layout: post
title:  "VLAN Tagging in Debian"
date:   2014-02-14 07:12:00
categories: linux
---
I run a Debian box at work with a Windows VM for some less-avoidable tasks. Due to some network restructuring, we now need to have our host and guest OSes in separate VLANs. This post briefly explains the process of accomplishing this task.

Keep in mind that your switch port will need to be trunked for this to work at all. I am by no means a network guy, so the network side of things is outside the scope of this post.

## Enable 802.1q
802.1Q is "the networking standard that supports Virtual LANs (VLANs) on an Ethernet network" ([Wikipedia](http://en.wikipedia.org/wiki/IEEE_802.1Q)). We need to load the 802.1Q kernel module to enable VLAN tagging.

In a terminal window, run the following command:

```
modprobe 8021q  
```

This will immediately load the 802.1Q kernel module. To be sure the module loads on startup, open `etc/modules` in a text editor and add this line:

```
8021q  
```

## Install VLAN package
This is a package that allows you to add VLAN devices to your physical devices.

apt-get install vlan  

## Configure network interfaces
This part can be easily be modified to fit your needs. I don't need my host OS to be able to access the same VLAN as my VM, so I don't have an IP assigned to the `eth0.10` interface.

Open `/etc/network/interfaces` and use this configuration, modified for your network:

```
# The loopback network interface
auto lo
iface lo inet loopback

# Physical adapter
auto eth0
iface eth0 inet manual
  up ifconfig eth0 0.0.0.0 up

# VLAN 10 interface for VMs; no IP
auto eth0.10
iface eth0.10 inet manual
  up ifconfig eth0.10 0.0.0.0 up

# VLAN 20 interface for host
auto eth0.20
iface eth0.20 inet static
  address *.*.*.*
  netmask 255.255.255.0
  gateway *.*.*.*
```

## Restart network interfaces

```
ifdown eth0
ifup eth0
ifup eth0.10
ifup eth0.20
```

You can now set your VMs to use the `eth0.10` interface. If all goes well, you should be able to access VLAN 10 from your VM and VLAN 20 from your host.

Reference: https://wiki.debian.org/NetworkConfiguration
