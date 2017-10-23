---
layout: post
title:  "UPDATED: All-in-One OpenShift Container Platform 3.6 Architecture"
date:   2017-10-19 11:11:01 -0400
categories: openshift ocp
---
# OpenShift Container Platform 3.6 Mini Cluster (for learning purposes only)

By Chris Kim (chris.kim@redhat.com) - Last Updated Oct 17, 2017, v1.0

NOTE: This is a WORK IN PROGRESS! I cannot guarantee that anything I say in here will work out of box for you! This is simply documenting my steps to create a semi-contained OpenShift infrastructure to familiarize yourself with deploying OpenShift.

If you have any questions feel free to e-mail me at [chris.kim@redhat.com](mailto:chris.kim@redhat.com) 

Note that these instructions can pretty easily be adapted to install Origin, but are written for installing OCP on registered RHEL boxes. 

Please try to understand what you're putting in the files I provide below! If you just copy and paste you won't learn anything! 

## High Level Steps

To configure a miniature openshift cluster, you'll have to do a few things. You'll need to configure a DNS forwarder (BIND in this example), configure an NFS server (instructions coming in a later document, so not covered now unfortunately), prepare the OpenShift nodes, and run the install. If everything goes according to plan you should be able to deploy a mini OpenShift cluster within 3 hours of starting this document.

MAKE SURE YOU SET THE SEARCH DOMAIN TO vm.example.com OR WHATEVER YOUR HOST DOMAIN IS ON OS INSTALL/BEFORE YOU RUN THE ANSIBLE PLAYBOOK, otherwise you will run into DNS resolving issues as OpenShift won't be able to resolve things like docker-registry.default.svc!!!!!

## VM Information

In order to install a semi-working vanilla cluster of OpenShift, you need a couple of VM's on a network that you control with static IP's. I have documented the basic VM's I create and will reference throughout this document below. I am assuming you are running RHEL 7.4.

If you follow the chart below, you will use 18.5 GB of ram on your host system. The recommended spec's are from the documentation for minimum specs. If you don't meet the minimum specs, you will have to disable checks, which we'll discuss below.

| VM Hostname | VM Description | CPU | Memory | Disks/Space | IP | Alternative Hostnames |
|:-----------:|:---------------|:---:|:------:|:-----------:|:--:|:---------------------:|
|dns.vm.example.com|This is a VM for hosting your local DNS forwarder|1 vCPU|512 MB|10 GB Disk|172.16.32.10|N/A|
|nfs.vm.example.com|This is a VM for hosting your persistent volumes|1 vCPU|2 GB|30 GB Disk|172.16.32.12|N/A|
|master.vm.example.com|This is your OpenShift Master + ETCD| 2 vCPU | 8 GB (16 GB Recommended) | 42 GB Disk | 172.16.32.15 | openshift.example.com |
|infranode.vm.example.com|This is your infrastructure node (runs metrics, logging, and router)| 1 vCPU | 6 GB (8 GB Recommended) | 16 GB disk for root FS, 16 GB disk for block storage for Docker (do not provision this disk on startup)|172.16.32.20|*.cloudapps.example.com|
|appnode.vm.example.com|This is your app node. You should have more than one, but for the smallest cluster, one is okay| 1 vCPU | 4 GB (8 GB Recommended) | 16 GB disk for root FS, 16 GB disk for block storage for Docker (do not provision this disk on startup)|172.16.32.21|N/A|


## Host Preparation

You have to prepare the hosts for docker block storage, but basically just follow the host prep directions in this document on all of the hosts (master, infranode, appnode)

<https://docs.openshift.com/container-platform/3.6/install_config/install/host_preparation.html>

This includes subscribing the system i.e. `subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-ose-3.6-rpms" \
    --enable="rhel-7-fast-datapath-rpms"`

## Ansible Hosts File

From your ansible node (which in this case can be master.vm.example.com), make sure you install the package `atomic-openshift-utils` which will include the required openshift-ansible playbooks. They will be located in `/usr/share/ansible/openshift-ansible/`

You will have to configure an Ansible hosts file (you can just modify `/etc/ansible/hosts`)

#### /etc/ansible/hosts

```
[OSEv3:children]
masters
etcd
nodes

[OSEv3:vars]
openshift_deployment_type=openshift-enterprise
openshift_release=v3.6

openshift_master_api_port=443
openshift_master_console_port=443
openshift_portal_net=172.30.0.0/16
osm_cluster_network_cidr=10.128.0.0/14

openshift_master_cluster_method=native
openshift_master_cluster_hostname=dashboard.example.com
openshift_master_cluster_public_hostname=dashboard.example.com

openshift_disable_check=memory_availability,disk_availability

openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
openshift_master_htpasswd_users={'admin': '$apr1$6CZ4noKr$IksMFMgsW5e5FL0ioBhkk/', 'developer': '$apr1$AvisAPTG$xrVnJ/J0a83hAYlZcxHVf1'}

[OSEv3:vars]

[masters]
master.vm.example.com

[etcd]
master.vm.example.com

[nodes]
master.vm.example.com openshift_node_labels="{'region': 'master'}"
infranode.vm.example.com openshift_node_labels="{'region': 'infra'}"
appnode.vm.example.com openshift_node_labels="{'region': 'primary'}"
```

## DNS Information for OpenShift Mini Cluster

In order to properly deploy OpenShift on your home network, you need to deploy a DNS server. Note that this should be a full blown DNS server, rather than DNSMASQ or any other DNS proxy services.

This will require a separate VM (although this can be the same VM you use to run NFS from, just remember the fact that if you run NFS you will need to scope the disk space of the VM a bit better.

#### Recommended Minimum Specs:

* Operating System: CentOS/RHEL 7
* HD Space: 10 Gb
* Ram: 512 Mb
* vCPU: 1
* Hostname: dns.vm.example.com (or dnsnfs.vm.example.com if you are running NFS from this VM as well)

### General Steps for DNS VM

Until I get around to performing this install from scratch, I will post pseudo steps to get a BIND DNS Server instance running for the purposes of configuring OpenShift.

 1. Install RHEL/CentOS
 2. Register RHEL instance (only applicable if RHEL) 
 3. `yum update`
 4. `yum install named`
 6. Configure `named` to both forward unknown DNS to upstream DNS servers and to allow anyone on your network to query it (See /etc/named.conf example below)
 7. Configure `example.com` DNS zone for `openshift.example.com` entry to access the dashboard (see /var/named/example.com.hosts)
 8. Configure `cloudapps.example.com` DNS zone for wildcard DNS (see /var/named/cloudapps.example.com.hosts)
 7. Configure `vm.example.com` DNS zone for your actual master, infra, and app node(s) (see /var/named/vm.example.com.hosts)
 8. `systemctl enable named`
 9. `systemctl start named`
 10. At this point, you should edit all of your nodes and set the DNS server to the IP of your DNS VM. (either through NetworkManager or through /etc/resolv.conf if NetworkManager is disabled)
 
#### /etc/named.conf
 
```
options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-enable yes;
        dnssec-validation yes;

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.iscdlv.key";

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
        forwarders {
                8.8.8.8;
                8.8.4.4;
        };
};

zone "example.com" {
	type master;
	file "example.com.hosts";
};

zone "cloudapps.example.com" {
	type master;
	file "cloudapps.example.com.hosts";
};

zone "vm.example.com" {
	type master;
	file "vm.example.com.hosts";
};
```

#### /var/named/example.com.hosts
```
$ttl 38400
example.com.	IN	SOA	dns.vm.example.com. youremail.example.com. (
			1506743187
			10800
			3600
			604800
			38400 )
example.com.	IN	NS	dns.vm.example.com.
openshift.example.com.	IN	A	172.16.32.15 ; This should point to your master
dns.vm.example.com.	IN	A	172.16.32.10 ; This should point to your DNS server
```

#### /var/named/cloudapps.example.com.hosts
```
$ttl 38400
cloudapps.example.com.	IN	SOA	dns.vm.example.com. youremail.gmail.com. (
			1506743256
			10800
			3600
			604800
			38400 )
cloudapps.example.com.	IN	NS	dns.vm.example.com.
*.cloudapps.example.com.	IN	A	172.16.32.20 ; This should be the IP of your infrastructure node
```

#### /var/named/vm.example.com.hosts

```
$ttl 38400
vm.example.com.	IN	SOA	dns.vm.example.com. youremail.gmail.com. (
			1506743066
			10800
			3600
			604800
			38400 )
vm.example.com.	IN	NS	dns.vm.example.com.
dns.vm.example.com.		IN	A	172.16.32.10 ; This should be the IP of your DNS VM
master.vm.example.com.	IN	A	172.16.32.15 ; This should be the IP of your master VM
infranode.vm.example.com.	IN	A	172.16.32.20 ; This should be the IP of your infrastructure VM
appnode.vm.example.com.	IN	A	172.16.32.21 ; This should be the IP of your app node VM
nfs.vm.example.com.		IN	A	172.16.32.12 ; This should be the IP of your NFS server (should you want persistence)
```

## Configure Passwordless SSH

You need to run `ssh-keygen` on your ansible node and then `ssh-copy-id master|infranode|appnode`

Set `host_key_checking = False` under the `[defaults]` section of your `/etc/ansible/ansible.cfg` file

## Running ANSIBLE!!

Once you've done these things, and you can run `ansible nodes -m ping` and get no red, you're ready to run the install!

`ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/byo/config.yml`
