# Ansible Automation Platform 2.x automated demo

## What is this?
This repository holds some kcli plans and Ansible playbooks to automatically build
a multi-node Ansible Automation Platform 2.x infrastructure on libvirt. Perfect for
demoing and playing around.

Only thing that you need to do is deploy the appropriate plan on your laptop
/ workstation.

## Pre-reqs
Before you can deploy the plan, you need to do a couple of things:
- clone this repo onto your local machine
- install kcli, see https://copr.fedorainfracloud.org/coprs/karmab/kcli/
- download rhel-8.4-x86_64-kvm.qcow2 from [access.redhat.com](https://access.redhat.com/downloads/content/479/ver=/rhel---8/8.4/x86_64/product-software) and drop it into `/var/lib/libvirt/images`.
- create appropriate `~/.kcli/config.yml` and `~/.kcli/profiles.yml` files. Find some
examples in the `samples` directory of the [kcli GitHub
repo](https://github.com/karmab/kcli/tree/master/samples).
- download ansible-automation-platform-setup-2.0.1-1-early-access.tar.gz from
  [access.redhat.com](https://access.redhat.com/downloads/content/480/ver=Early%20Access%202.0/rhel---8/Early%20Access%202.0/x86_64/product-software) and drop it into the `files` directory of this repo. The download location will 
  change after AAP 2.1 is released properly.

## Running the plan
After filling in the pre-requisites - which you have to do only once per machine - you
can deploy the plan from the root of this project by running:
```
kcli create plan aap-cluster -f plans/aap2_cluster.yml
```

Wait for a couple of minutes for your plan to spin up and you are done :)

## Cleaning up
As you are running RHEL8 images, when after running `kcli delete plan aap-cluster`,
you'll need to manually remove the subscription allocations on access.redhat.com.

## Pull requests, feature requests, etc.
Please send them :)
