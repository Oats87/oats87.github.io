---
layout: post
title:  "OpenShift Container Platform 3.6 on Single Hypervisor Guide"
date:   2017-10-19 11:11:01 -0400
categories: openshift ocp
---
# OpenShift Container Platform 3.6 Mini Cluster (for learning purposes only)

By Chris Kim (me@chrishkim.com) - Last Updated Mar 9, 2018, v1.6

NOTE: This is a WORK IN PROGRESS! I cannot guarantee that anything I say in here will work out of box for you! This is simply documenting my steps to create a semi-contained OpenShift infrastructure to familiarize yourself with deploying OpenShift.

If you have any questions feel free to e-mail me at [me@chrishkim.com](mailto:me@chrishkim.com)

Note that these instructions can pretty easily be adapted to install Origin, but are written for installing OCP on registered RHEL boxes.

Please try to understand what you're putting in the files I provide below! If you just copy and paste you won't learn anything!

## Known Issues:


## High Level Steps

To configure a miniature openshift cluster, you'll have to do a few things. You'll need to configure a DNS forwarder (BIND in this example), configure an NFS server, prepare the OpenShift nodes, and run the install. If everything goes according to plan you should be able to deploy a mini OpenShift cluster within 3 hours of starting this document.

MAKE SURE YOU SET THE SEARCH DOMAIN TO vm.example.com OR WHATEVER YOUR HOST DOMAIN IS ON OS INSTALL/BEFORE YOU RUN THE ANSIBLE PLAYBOOK, otherwise you will run into DNS resolving issues as OpenShift won't be able to resolve things like docker-registry.default.svc!

## VM Information

In order to install a semi-working vanilla cluster of OpenShift, you need a couple of VM's on a network that you control with static IP's. I have documented the basic VM's I create and will reference throughout this document below. I am assuming you are running RHEL 7.4.

If you follow the chart below, you will use 19.5 GB of ram on your host system. The recommended spec's are from the documentation for minimum specs. If you don't meet the minimum specs, you will have to disable checks, which we'll discuss below.

| VM Hostname | VM Description | CPU | Memory | IP | Alternative Hostnames |
|:-----------:|:---------------|:---:|:------:|:--:|:---------------------:|
|ansible.vm.example.com|Ansible VM for running install/upgrade|1 vCPU|1024 MB|172.16.32.5|N/A|
|dns.vm.example.com|This is a VM for hosting your local DNS forwarder|1 vCPU|512 MB|172.16.32.10|N/A|
|nfs.vm.example.com|This is a VM for hosting your persistent volumes|1 vCPU|2 GB|172.16.32.12|N/A|
|master.vm.example.com|This is your OpenShift Master + ETCD| 2 vCPU | 8 GB (16 GB Recommended) | 172.16.32.15 | openshift.example.com |
|infranode.vm.example.com|This is your infrastructure node (runs metrics, logging, and router)| 1 vCPU | 6 GB (8 GB Recommended) |172.16.32.20|*.cloudapps.example.com|
|appnode.vm.example.com|This is your app node. You should have more than one, but for this mini cluster, one is okay| 1 vCPU | 4 GB (8 GB Recommended) | 172.16.32.21|N/A|

#### Storage Recommendations

| VM Short Hostname | Storage |
|:-----------------:|:-------:|
|ansible|10 GB Disk|
|dns|10 GB Disk|
|nfs|42 GB Disk|
|master|42 GB Disk for root FS, 16 GB disk for Docker|
|infranode|16 GB disk for root FS, 16 GB disk for Docker|
|appnode|16 GB disk for root FS, 16 GB disk for Docker|


Please note: Do not provision/use the 16 gb disk on the master/infra/appnodes on install, this disk will be configured for use with Docker for the block storage.

## Host Preparation

You have to prepare the hosts for docker block storage, but basically just follow the host prep directions in this document on all of the hosts (master, infranode, appnode)

I have outlined the steps below, and which hosts the steps are applicable to:

1. (ansible, dns, nfs, master, infranode, appnode) Run `subscription-manager register` to register the system with Red Hat so that you can use the Red Hat repositories

2. (ansible, master, infranode, appnode) Run `subscription-manager attach --pool={POOL_ID}` to attach the system to a valid subscription that has OpenShift Container Platform on it, alternatively if you only have one pool, you should attach your DNS and NFS nodes to this as well.

3. (dns, nfs) Run `subscription-manager attach --pool={POOL_ID}` to attach your NFS and DNS servers to the pool ID of your general RHEL 7 subscription. This is only necessary if you didn't attach it already in the step above.

4. (ansible, dns, nfs, master, infranode, appnode) Run `subscription-manager repos --disable="*"` to disable all repositories except the ones you need

5. (dns, nfs) Run `subscription-manager repos --enable="rhel-7-server-rpms" --enable="rhel-7-server-extras-rpms"` to enable the RHEL 7 Server RPMs

6. (ansible, master, infranode, appnode) Run `subscription-manager repos --enable="rhel-7-server-rpms" --enable="rhel-7-server-extras-rpms" --enable="rhel-7-server-ose-3.6-rpms" --enable="rhel-7-fast-datapath-rpms"`

7. (ansible) Run `yum install atomic-openshift-utils`

8. (master, infranode, appnode) Run `yum install docker-1.12.6`

9. (master, infranode, appnode) Modify `/etc/sysconfig/docker-storage-setup` to reflect that 16 GB disk you created. This could be vdb, sdb, sdc, etc. so make sure you find the right disk.

## Configure Passwordless SSH

You need to run `ssh-keygen` on your ansible node and then `ssh-copy-id master.vm.example.com; ssh-copy-id infranode.vm.example.com; ssh-copy-id appnode.vm.example.com`

Set `host_key_checking = False` under the `[defaults]` section of your `/etc/ansible/ansible.cfg` file

#### /etc/sysconfig/docker-storage-setup
```
DEVS=/dev/vdb
VG=docker-vg
```

10. (master, infranode, appnode) Run `docker-storage-setup` to provision the LVM for that 16 gb disk
11. Enable and start docker to so the OpenShift installer doesn't fail: `systemctl enable docker; systemctl start docker`

You should be ready to run the install at this point, other hosts may need to be configured but that will be covered below.

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
# Set the ansible_ssh_user.
ansible_ssh_user=root

# If you are not able to log into each machine as root and must sudo instead, the lines should be:
# ansible_ssh_user=ocpadmin
# ansible_become=true

# Specify deployment as OpenShift Container Platform. If you are deploying Origin this should be set to Origin
openshift_deployment_type=openshift-enterprise

# Specify the OpenShift version. This document is set up for 3.6 currently.
openshift_release=v3.6

# Set the API and Web Console port to 443 (vs. the normal 8443)
openshift_master_api_port=443
openshift_master_console_port=443

# Specify the default portal net (used for pods)
# and cluster network (used for services)
openshift_portal_net=172.30.0.0/16
osm_cluster_network_cidr=10.128.0.0/14

# Set the cluster method as native
openshift_master_cluster_method=native

# Set the cluster hostname and public cluster hostname
openshift_master_cluster_hostname=openshift.example.com
openshift_master_cluster_public_hostname=openshift.example.com

# Set the default wildcard DNS, so your applications will be created at
# docker-registry-console-default.cloudapps.example.com for example
openshift_master_default_subdomain=cloudapps.example.com

# Disable health check(s) - Covered below
openshift_disable_check=memory_availability

# Configure simple HTPasswd authentication provider
# The default credentials will be {admin:adm-password} and {developer:devel-password}
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
openshift_master_htpasswd_users={'admin': '$apr1$6CZ4noKr$IksMFMgsW5e5FL0ioBhkk/', 'developer': '$apr1$AvisAPTG$xrVnJ/J0a83hAYlZcxHVf1'}

# Ansible will set the insecure registry parameter in each node to this if you set this, it should be what is contained in the openshift_portal_net variable
openshift_docker_insecure_registries="172.30.0.0/16"

[masters]
master.vm.example.com

[etcd]
master.vm.example.com

[nodes]
master.vm.example.com openshift_node_labels="{'region': 'master'}"
infranode.vm.example.com openshift_node_labels="{'region': 'infra'}"
appnode.vm.example.com openshift_node_labels="{'region': 'primary'}"
```

## Disabling Certain Health checks

You'll notice in the `/etc/ansible/hosts` file there was a line that said `openshift_disable_check`. This line is used to specify which health checks you want to disable. For the mini-cluster, provided you don't hit the minimum memory availability, you'll need to disable `memory_availability`. If you have more than just memory as your restriction, you may want to add addition check disables such as `disk_availability` and `docker_storage`, in a comma delimited list i.e. `openshift_disable_check=memory_availability,disk_availability,docker_storage`

## Tune Ansible for Performance

You'll probably want to configure Ansible to be a little faster than it comes out of the box. To do this, you should enable pipelining (it will reuse the same SSH session) and increase the number of forks (only matters if you have > 5 nodes total, particularly for HA installs of 3-3-3)

#### /etc/ansible/ansible.cfg

```
[defaults]
forks=20
pipelining=True
```

## Verify Ansible Communication

Once you've configured your Ansible Hosts file, you'll probably want to make sure Ansible can actually log in and touch all of your nodes. To do this, you should simply run the Ansible ping ad-hoc command: `ansible nodes -m ping` You'll want to see all green. If you see red, correct the issue and try the command again.

You will most likely run into Ansible asking if you want to trust the host, in this case, just keep typing `y` and hitting Enter until you are returned to the prompt. Once you are returned to the prompt, you can run ping again and it should not complain anymore.

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
 10. At this point, you should edit all of your nodes and set the DNS server to the IP of your DNS VM. (i.e. `DNS1=172.16.32.5` in `/etc/sysconfig/network-scripts/ifcfg-eth0`)

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

        dnssec-enable no;
        dnssec-validation no;

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
bastion.vm.example.com. IN  A 172.16.32.5 ; This is the IP of your Bastion host
dns.vm.example.com.		IN	A	172.16.32.10 ; This should be the IP of your DNS VM
master.vm.example.com.	IN	A	172.16.32.15 ; This should be the IP of your master VM
infranode.vm.example.com.	IN	A	172.16.32.20 ; This should be the IP of your infrastructure VM
appnode.vm.example.com.	IN	A	172.16.32.21 ; This should be the IP of your app node VM
nfs.vm.example.com.		IN	A	172.16.32.12 ; This should be the IP of your NFS server (should you want persistence)
```

## Configure NFS server for registry, logging, metrics

The general steps to configuring an NFS server for use with OpenShift are quite simple. I've listed the steps to do so below:

1. `yum install nfs-utils`
2. `mkdir /var/nfs; mkdir /var/nfs/logging; mkdir /var/nfs/registry; mkdir /var/nfs/metrics`
3. `chown -R nfsnobody:nfsnobody /var/nfs`
4. `chmod -R 777 /var/nfs`
5. Modify the `/etc/exports` file to allow mounting of `/var/nfs/{logging|registry|metrics}`, i.e. the following /etc/exports file:

#### /etc/exports
```
/var/nfs/registry 172.16.32.0/24(rw,root_squash)
/var/nfs/metrics 172.16.32.0/24(rw,root_squash)
/var/nfs/logging 172.16.32.0/24(rw,root_squash)
```

6. Start/enable the NFS server (if it hasn't been started already): `systemctl enable nfs; systemctl start nfs`
7. Export the new shares if they haven't already been exported: `exportfs -a`
8. `firewall-cmd --permanent --add-service=nfs`
9. `firewall-cmd --reload`
10. Test mounting the NFS shares from any node in the network, i.e. `mkdir /mnt/nfstest; mount -t nfs nfs.vm.example.com:/var/nfs/registry /mnt/nfstest; touch /mnt/nfstest/test`
11. On the NFS server, check that the new `test` file exists: `ls /var/nfs/registry`
12. Back on the node that you mounted the NFS share on, `rm -rf /mnt/nfstest/test; umount /mnt/nfstest`

You should be good to go at this point for deploying using persistent storage

## Adding NFS Storage Lines to /etc/ansible/hosts

In order to enable OpenShift and Ansible to know that you are using NFS for persistent storage, you will have to add a few lines below the `[OSEv3:vars]` group in your Ansible hosts file

#### /etc/ansible/hosts ADDITION

```
openshift_hosted_registry_storage_kind=nfs
openshift_hosted_registry_storage_access_modes=['ReadWriteMany']
openshift_hosted_registry_storage_host=nfs.vm.example.com
openshift_hosted_registry_storage_nfs_directory=/var/nfs
openshift_hosted_registry_storage_volume_name=registry
openshift_hosted_registry_storage_volume_size=10Gi

openshift_hosted_metrics_deploy=true
openshift_hosted_metrics_storage_kind=nfs
openshift_hosted_metrics_storage_access_modes=['ReadWriteOnce']
openshift_hosted_metrics_storage_host=nfs.vm.example.com
openshift_hosted_metrics_storage_nfs_directory=/var/nfs
openshift_hosted_metrics_storage_volume_name=metrics
openshift_hosted_metrics_storage_volume_size=10Gi

openshift_hosted_logging_deploy=true
openshift_hosted_logging_storage_kind=nfs
openshift_hosted_logging_storage_access_modes=['ReadWriteOnce']
openshift_hosted_logging_storage_host=nfs.vm.example.com
openshift_hosted_logging_storage_nfs_directory=/var/nfs
openshift_hosted_logging_storage_volume_name=logging
openshift_hosted_logging_storage_volume_size=10Gi
```

## Running Ansible

Once you've done these things, and you can run `ansible nodes -m ping` and get no red, you're ready to run the install!

`ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/byo/config.yml`

After the install completes, provided you don't have any issues with deployment, you should be able to access your cluster at

[https://openshift.example.com](https://openshift.example.com) if you set your machine DNS to the DNS of your internal DNS server (that is overriding your DNS setup)

## Running Health Checks

If you wish to run health checks after your install (and given you didn't disable all of them using the `openshift_disable_check` variable), you can invoke a manual run of just the health checks by running:

`ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-checks/health.yml`

# No Longer Relevant Known Issues

### ElasticSearch Master Deployment Not Ready

#### UPDATE

This has been fixed by the latest Z-Stream in OpenShift 3.6. You should no longer encounter this issue.

When you deploy openshift-logging currently, we are experiencing a bit of an issue where the ElasticSearch master's are running an older image version that doesn't support the latest readiness checks/discovery algorithm. This requires you to revert back, hence you should disable the readiness probe (`oc edit dc logging-es-data-master-...` and remove the section that has `readinessProbe:`) and revert back to the master discovery algorithm (`oc edit cm logging-elasticsearch` to change:

```
cloud:
   kubernetes:
     pod_label: ${POD_LABEL}
     pod_port: 9300
     namespace: ${NAMESPACE} 
```
to
```
cloud:
   kubernetes:
     service: ${SERVICE_DNS}
     namespace: ${NAMESPACE}
```

Please keep in mind white space when you're performing this fix, as YAML is very sensitive to it.

Thank you to [@wozniakjan](https://github.com/wozniakjan) for this solution found [here](https://github.com/openshift/openshift-ansible/issues/5497#issuecomment-331372471)

### FluentD Misconfigured Config Map

#### UPDATE

This has been fixed by the latest version of OpenShift 3.6. You should no longer run into this issue, and you should also revert your ConfigMap back to the original format, so you perform proper tagging and labelling of logs within the EFK stack (which also helps to reduce clutter and load)

FluentD currently has the wrong config map with it, which is causing issues with 3.6 logging deployment. If you look at the logs of the FluentD containers, you will see `journal.system` warnings. To resolve this, you must edit the configmap (`oc edit configmap/logging-fluentd`) from

```
   <label @INGRESS>
   ## filters
     @include configs.d/openshift/filter-pre-*.conf
     @include configs.d/openshift/filter-retag-journal.conf
     @include configs.d/openshift/filter-k8s-meta.conf
     @include configs.d/openshift/filter-kibana-transform.conf
     @include configs.d/openshift/filter-k8s-flatten-hash.conf
     @include configs.d/openshift/filter-k8s-record-transform.conf
     @include configs.d/openshift/filter-syslog-record-transform.conf
     @include configs.d/openshift/filter-viaq-data-model.conf
     @include configs.d/openshift/filter-post-*.conf
   ##
   </label>
   <label @OUTPUT>
   ## matches
     @include configs.d/openshift/output-pre-*.conf
     @include configs.d/openshift/output-operations.conf
     @include configs.d/openshift/output-applications.conf
     # no post - applications.conf matches everything left
   ##
   </label>
```
to
```
   <label @INGRESS>
   ## filters
     @include configs.d/openshift/filter-pre-*.conf
     @include configs.d/openshift/filter-retag-journal.conf
     @include configs.d/openshift/filter-k8s-meta.conf
     @include configs.d/openshift/filter-kibana-transform.conf
     @include configs.d/openshift/filter-k8s-flatten-hash.conf
     @include configs.d/openshift/filter-k8s-record-transform.conf
     @include configs.d/openshift/filter-syslog-record-transform.conf
     @include configs.d/openshift/filter-viaq-data-model.conf
     @include configs.d/openshift/filter-post-*.conf
   ##
   ## matches
     @include configs.d/openshift/output-pre-*.conf
     @include configs.d/openshift/output-operations.conf
     @include configs.d/openshift/output-applications.conf
     # no post - applications.conf matches everything left
   ##
   </label>
```

Thank you to [Louis Santillan](https://github.com/lpsantil) for providing the solution for this issue! 