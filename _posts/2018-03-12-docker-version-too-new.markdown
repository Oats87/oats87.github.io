---
layout: post
title:  "OpenShift Container Platform 3.6/3.7 Installation/Upgrade Failure due to docker 1.13.1 existence"
date:   2018-03-12 09:33:00 -0800
categories: openshift ocp
---
# OpenShift Container Platform 3.6/3.7 Installation/Upgrade Failure due to docker-1.13.1 existence

By Chris Kim (me@chrishkim.com)

If you have any questions feel free to e-mail me at [me@chrishkim.com](mailto:me@chrishkim.com)

## Introduction

Recently, I have been working on automating installations of OpenShift Container Platform on Red Hat OpenStack 12, and I encountered a quite annoying issue during installation. As Red Hat has recently released docker-1.13.1, there are issues popping up with the Ansible installer checking too-new versions of Docker. 

## How to fix this?

To be honest, the fix for this issue is quite simple (although rudimentary at best), it is entirely a work around rather than a permanent solution.

The quick fix is to add

```
openshift_disable_check=package_version
```

to your Ansible Hosts File, under your conventional `[OSEv3:vars]` section.

The long-term fix relates work with Red Hat Engineers and QE; bugs have been opened about this issue [BZ 1551862](https://bugzilla.redhat.com/show_bug.cgi?id=1551862) and a pull request has already been created to fix this [PR 7347](https://github.com/openshift/openshift-ansible/pull/7347). I would expect this to be out within the next 2 Z-Streams.
