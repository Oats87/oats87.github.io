---
layout: post
title:  "OpenShift Container Platform 3.6/3.7 Integration with OpenStack Cloud Provider on a Provider Network Notes"
date:   2018-03-09 04:00:00 -0800
categories: openshift ocp
---
# OpenShift Container Platform 3.6/3.7 on OpenStack using Cloud Provider on a Flat Network

By Chris Kim (chris.kim@redhat.com) - Last Updated Mar 9, 2018, v1.0

If you have any questions feel free to e-mail me at [me@chrishkim.com](mailto:me@chrishkim.com) or officially at Red Hat via [chris.kim@redhat.com](mailto:chris.kim@redhat.com)

## Introduction

Recently, I was tasked with installing OpenShift on an OpenStack environment provided by a customer. Their goal was to increase automation of their OpenShift environments, and be able to automate the creation/deletion of environments. Thus, to provide a programmable compute layer, they utilized Red Hat OpenStack.

The only difference was rather than using an SDN themselves, the customer had configured their compute layer to simply bridge the instance NIC's to their internal network (thus their instances were no different from a conventional VM)

## Problems Encountered

Really, the only problem that was encountered on the installation of OpenShift on OpenStack with CloudProvider integration was the nodeName settings. Thus, what was occurring was two problems, encountered independently.

1. The Node Name (and thus OpenStack Instance Name) must be RFC-1123 compliant, meaning it needs to be a real DNS name. 
2. The Node Name needs to be resolvable, or else you will run into issues when an AOS Node starts for the first time.

## What is the real problem here?

OpenShift on OpenStack with CloudProvider integration is set up in such a way that on the startup of an OpenShift node, the cloud provider configuration polls the OpenStack API to pull the actual instance name of the node. Once it pulls this instance name, it sets the node name to the instance name (thus overriding whatever you had in your ansible hosts file). There is no known workaround for this.

The issue that was occurring was due to a mismatch of node names for the OVS SDN (whether it was subnet/multitenant/networkpolicy). The Ansible Installer would create hostnetworks for the nodes given the instance name, but when the node started, it would try to get the hostnetwork of the file configured node name (in this case IP Address), which doesn't exist anywhere else. This would make it so that the SDN could never be initialized and the node service could never start. 

## Issue 1: RFC-1123 Compliance

This one is pretty self explanatory; but an OpenShift Node Name must be RFC-1123 compliant (due to technical reasons and the nature of how OpenShift/Kubernetes is built). Thus, if you are using special characters in your node names, you will likely run into issues, as the node will not be able to start due to the non-compliance.

## Issue 2: Resolvability

When an OpenShift Node is installed, the node name must be resolvable (or at least set to something else) such that the Ansible Installer is able to see a resolution for the node name. If that doesn't exist, the Ansible Installer will fall back to using the IP address which in the event of an OpenStack integration is incorrect.

## Resolution:

The final resolution that we applied was the following:

1. The Instance Names were set to FQDN's of the nodes, that actually resolved via DNS
2. The line for each node in the Ansible Hosts file had an `openshift_hostname=` section that was set to the FQDN. Thus, a sample line would be like the following:

`mynode.fqdn openshift_hostname=mynode.fqdn openshift_node_labels="{'region': 'primary'}"` 