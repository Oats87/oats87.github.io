---
layout: post
title:  "RHEL 7 Virtual Machine (libvirt) Bridge Configuration"
date:   2017-10-19 20:32:01 -0400
categories: rhel vm
---
# Configuring a RHEL 7 Machine to bridge eth0 to allow the host to access the VM's running on it

## Why?

When you use macvtap with a RHEL box with hosted virtual machines (through libvirt/kvm) you are not able to access the virtual machines running on the machine. This makes the host machine useless if you plan on using it as a proxy system to access the virtual machines.

## When?

This should be done if you have a single interface going into your machine (or only want to use a single interface) and plan on using the machine as a proxy to allow you to manage all of the virtual machines on it. My use case for this scenario was my virtual host served as a host and ansible control node for a test OpenShift cluster I ran on it. This machine had it's port 22 exposed to the internet to allow me to access it remotely.

## Requirements?

* NetworkManager
* Know the name of your network interface
* Set a static IP for your interface, I'm pretty sure you can set your interface to DHCP as well but I haven't done this yet. 

## How?

Magic. But actually, I configured this through two files in the `/etc/sysconfig/network-scripts` folder. I've listed them below

### /etc/sysconfig/network-scripts/ifcfg-br0

```
TYPE=Bridge
BOOTPROTO=static
DEVICE=br0
ONBOOT="yes"
IPADDR="172.16.32.5"
PREFIX="24"
GATEWAY="172.16.32.1"
DNS1="172.16.32.10" # In my cluster, I had a DNS forwarder I used for OpenShift DNS names. I wanted to be able to ping DNS hostnames from the host so I set my primary DNS server as this.
DNS2="8.8.8.8"
DOMAIN="vm.example.com" # This let me set the search domain for my virtual machines so I could shorthand access them
```

### /etc/sysconfig/network-scripts/ifcfg-eth0

```
TYPE="Ethernet"
BOOTPROTO=static
NAME="eth0"
UUID="<INSERT THE UUID OF YOUR DEVICE HERE>"
DEVICE="eth0"
ONBOOT="yes"
BRIDGE=br0
```
