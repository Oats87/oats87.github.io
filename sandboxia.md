# Proposal: Sandboxia Labs for Red Hat Consulting West

## Overview

A request for funding to help expand the current Sand Box Labs in Red Hat Sunnyvale is being submitted to help enable our Red Hat Consulting West consultants to better enable themselves for customer engagements without worrying about hardware overhead or constraints in terms of a networking environment. The goal of this environment is to reduce the ramp up time of West Consultants, thus enabling them to be ready for engagements (particularily new college hire consultants). In addition, this lab will serve as a sandbox for West Consultants on-site, in the scenario that they require a mock environment to test certain scenarios that our customers have put into question.

## Benefits

This environment is beneficial in that it would allow West Consultants to work and learn with Red Hat products not normally usable on their local machines. While local virtual machines work in certain cases, there are many products that cannot easily be spun up (whether due to lack of local resources or lack of networking capabilities) on their local machines. This environment aims to help prevent that issue from occuring by providing a dedicated learning environment where west consultants are able to spin up virtual machines without a lot of thought or concern into the overhead required for the machines.

Although there are other Red Hat resources with similar goals for consultant enablement (for example OpenTLC), none of these environments allow the user to properly work with the infrastructure side of our products. For example, OpenTLC provides a "template" for deploying OpenShift Container Platform, but will only allow the user access to the container platform rather than the actual RHEL  nodes. This limits consultant enablement (especially in terms of learning the infrastructure side with troubleshooting) as this environment touts a "easy-to-use" environment, whereas most client sites encounter bugs that are only solved through real world simulative environments.

By allowing consultants to start from "ground zero" to deploy our products, they can simulate common client architectures or specific environment scenarios to help reproduce bugs for problem solving. For example, if a client is using FIPS compliant machines, it is possible to start a FIPS compliant RHEL box on the RHEVM cluster that would allow the consultant to work with the associated bugs that disabling the MD5 hash algorithm provides.

## Proposed Partnership with SCE

A proposed partnership is in works with the Red Hat SCE West team. This proposed partnership would allow both SCE West and RHC West to essentially "combine forces" to combine hypervisor and storage capabilities to create a larger and more robust sand box environment.

SCE West already has an established mini hypervisor lab where they currently have RHEV hypervisors. The proposed lab which will be built out of the partnership will utilize the 40 TB shared storage array with the additional hypervisors to essentially allow a large shared area.

This has been discussed with the points of contacts listed below:

### SCE West Team Contacts
| Contact Name   | Contact Position                     | Contact E-Mail      |
|:---------------|:-------------------------------------|:--------------------|
| Kevin Toyama   | Manager, SCE West                    | ktoyama@redhat.com  |
| Bryan Yount    | Senior Technical Account Manager     | byount@redhat.com   |
| Chip Shabazian | Senior Technical Account Manager (?) | cshabazi@redhat.com | 

### Red Hat Consulting Team Contacts

| Contact Name   | Contact Position                     | Contact E-Mail |
|:---------------|:-------------------------------------|:---------------|
| Chris Kim      | Associate Consultant                 | chrkim@redhat.com |
| Dmitry Shevrin | Senior Consultant                    | dshevrin@redhat.com |
| Louis Santillan | Senior Consultant                   | lsantill@redhat.com |
| Chris Timberlake | Consultant                         | ctimberl@redhat.com |

## Red Hat Technologies Used

The following technologies are being used to enable the West Labs environment:

* Red Hat Virtualization Manager
* Red Hat Enterprise Linux
* HP Servers
* Dell Switches (maybe?)

## Budget Breakdown

| Item Name | Estimated Cost | Quantity | Reasoning |
|:---------:|:---------------|:--------:|:----------|
| 100/1000 Managed Ethernet Switch | $150 | 1 | This is to enable all of the hosted hypervisors as well as storage arrays be manageable from remote locations. This is to primarily enable our remote consultants to access the physical machines should they require tweaking/management without the ability to be on-site in Sunnyvale (as most of our consulting administrative volunteer effort will be remote) |
| 10 Gbps Fiber Channel Card | $125 each | 4x | This is to allow all hosted hypervisors to connect to the 40 TB shared storage (required for RHEV)
| Hypervisor Server | $350 | 1 | This is to allow expansion of the current lab resources to enable our consultants to not run into resource overhead when sharing the environment with the SCE team as well as other consultants.

| Total Cost |
|:-----------|
| $1000      |

With the market of used servers being volatile, a specific server model cannot be specified. If the budget for this project is approved, the best server within the budget will be chosen at the time of purchase.

## Administrative Effort

Administration of this cluster will be on a sole volunteer basis, provided by a combination of work forces from the SCE and Consulting teams. Thus, by referencing the above tables of points of contact, it is possible to determine the initial administration team.

## Drawbacks/Concerns

Given that this lab will be voluntarily administered and hosted out of the Sunnyvale PnT Lab, this means that the lab will not be provided with any guarantee of uptime or reliability. Thus, given that all of the hardware for this lab is used hardware, there is no guarantee from the original manufacturer of said hardware to ensure reliability. This will be mitigated as best as possible by expanding the lab out with known reliable (in terms of statistical percentage) series of hardware. That being said, a best-effort-today strategy will be put in place to ensure as much uptime as possible.

Given the recent move from the Red Hat Mountain View Office, the infrastructure of the PnT lab has theoretically been improved. However, this does not absolve the office from brief power outages which would shut down the entire cluster temporarily.