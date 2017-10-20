---
layout: post
title:  "Using unetbootin on a RHEL/RHEL-like (Fedora, CentOS) environment"
date:   2017-10-20 11:00:01 -0400
categories: rhel general
---
# How to use unetbootin on a RHEL/RHEL-like environment

I've been using unetbootin to create bootable USB drives for longer than I can remember. When I started using Fedora 26 as my primary work operating system, I ran into a few issues that I've documented the solutions for below.

# Allowing unetbootin to run with escalated privileges

Once you install unetbootin, you have to run it with escalated privileges like such:

`sudo QT_X11_NO_MITSHM=1 /usr/bin/unetbootin`

The issue with this is that when you run an X program under escalated privileges, the escalated user (in this case root) is not allowed to hook to your session. This is solved through temporarily enabling that connection, with the following command:

`xhost local:root`

Thus, the final solution to allowing unetbootin to run with escalated privileges is to use the following snippet:

```
xhost local:root
sudo QT_X11_NO_MITSHM=1 /usr/bin/unetbootin
```

# Partitioning a flash drive for use with unetbootin

Let's say you have a brand new flash drive (at `/dev/sdb`) that has a weird partitioning that you want to use for unetbootin. In this case, we'll put a RHEL 7 ISO on the flash drive.

`sudo fdisk /dev/sdb`

If there are partitions on the drive, then you should run `d` to delete the partitions. Once you have an empty drive, run the following commands (within fdisk)

```
n # New Partition
p # Primary Partition Type
<ENTER> # Accept Default Partition Number (should be 1)
t # Set Type of Partition
b # WIN95FAT
a # Set partition as bootable
1 # Specify first partiton as bootable
w # Write changes to disk
```

After this, you may get a message if the flash drive was already mounted. In this case, you should run

`sudo partprobe /dev/sdb`

