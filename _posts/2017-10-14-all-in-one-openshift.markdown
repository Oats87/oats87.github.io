---
layout: post
title:  "Mini OpenShift Notes!"
date:   2017-10-14 11:11:01 -0400
categories: openshift ocp
---
# OCP Test on Virtualization

## THESE ARE NOT THE LATEST MINI OPENSHIFT NOTES!

## Setting Up OCP 3.5/3.6 on a single hosted virtualized environment

Things to keep in mind:
* Memory limits
* DNS availability
* Network routing
* Persistent volumes don't work

## What?

I set up an OpenShift Container Platform 3.6 environment on a single box running a hypervisor. This required a number of machines as I wanted to use the "Advanced Installation" process provided by OpenShift

## What Doesn't Work

Persistent storage/volumes.

## Provisioning hosts
Make sure that you can sudo without password on each machine, as well as providing the ability to log in without password. Thus, use `ssh-keygen` on the jump host then `ssh-copy-id` to each machine as ocpadmin.

For sudoing without a password, on each machine, change `/etc/sudoers` such that it looks like:
```
## Allows people in group wheel to run all commands
# %wheel	ALL=(ALL)	ALL

## Same thing without a password
%wheel	ALL=(ALL)	NOPASSWD: ALL
```

Essentially you are enabling NOPASSWD.

## VM Details

These are very specific for my environment, but they provide a little bit of perspective on how you would set up your environment should you want to do something similar to this.

| Machine Hostname                         | Machine IP   | Machine Type    | Machine Description                              |
|:-----------------------------------------|:-------------|:----------------|:-------------------------------------------------|
| os-workstation.rhlabenv.chrishkim.com    | 172.27.10.99 | RHEL 7.3 Server | Ansible Jump Host/Environment Access Machine     |
| os-firewall.rhlabenv.chrishkim.com       | 172.27.10.1  | pfSense         | Router/Firewall for OCP 3.6 Environment          |
| os-infrastructure.rhlabenv.chrishkim.com | 172.27.10.5  | RHEL 7.3 Server | DNS Server                                       |
| os-master.rhlabenv.chrishkim.com         | 172.27.10.25 | RHEL 7.3 Server | OCP 3.6 Master Host                              |
| os-infranode1.rhlabenv.chrishkim.com     | 172.27.10.50 | RHEL 7.3 Server | OCP 3.6 Infrastructure Node (marked region=infra)|
| os-appnode1.rhlabenv.chrishkim.com       | 172.27.10.75 | RHEL 7.3 Server | OCP 3.6 Application Node                         |

## Port Forwarding for Out-Of-Network Access

I have forwarded port 8443 to `os-master` and 443 and 80 to `os-infranode1`. This allows the machine to

## /etc/ansible/hosts

```
[OSEv3:vars]
openshift_master_identity_providers=[{'name': 'htpasswd_auth','login': 'true', 'challenge': 'true','kind': 'HTPasswdPasswordIdentityProvider','filename': '/etc/origin/master/htpasswd'}] # Sets up htpasswd authentication rather than the default denyall so that you can actually log in
openshift_master_cluster_hostname=ocpd.airlias.com # Cluster name
openshift_master_cluster_public_hostname=ocpd.airlias.com # Public hostname for cluster
openshift_master_default_subdomain=openshift.airlias.com # Provides the subdomain for new routes (an example subdomain would be test.openshift.airlias.com)
openshift_docker_insecure_registries=true # Configures Docker to allow access to insecure registries on 172.30.0.0/16
deployment_type=openshift-enterprise # Installs OpenShift Container Platform (rather than Origin)
openshift_disable_check=memory_availability # This tells Ansible to not check the availble memory. Very useful if your machines are under the recommended limit
ansible_ssh_user=ocpadmin # Tells ansible not to run as root
ansible_become=true # Tells Ansible to essentially use "sudo"
containerized=false # Use RPM Installation
openshift_hosted_metrics_deploy=true # Deploys ephemeral metrics


[OSEv3:children]
masters
nodes
etcd

[masters]
os-master.rhlabenv.chrishkim.com

[etcd]
os-master.rhlabenv.chrishkim.com

[nodes]
os-master.rhlabenv.chrishkim.com openshift_node_labels="{'region': 'master'}"
os-infranode1.rhlabenv.chrishkim.com openshift_node_labels="{'region': 'infra'}"
os-appnode1.rhlabenv.chrishkim.com openshift_node_labels="{'region': 'primary'}"
```

# Running Ansible

Your jump host should have the package `atomic-openshift-utils` installed. Once installed, you can run the ansible playbook through

`ansible-playbook -i /etc/ansible/hosts /usr/share/ansible/openshift-ansible/playbooks/byo/config.yml`
