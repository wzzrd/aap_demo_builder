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
- go to
    [access.redhat.com](https://access.redhat.com/management/subscription_allocations), create a subscription allocation and download it. Place the manifest.zip file in the `files` directory of this repo.
- download ansible-automation-platform-setup-2.0.1-1-early-access.tar.gz from
  [access.redhat.com](https://access.redhat.com/downloads/content/480/ver=Early%20Access%202.0/rhel---8/Early%20Access%202.0/x86_64/product-software) and drop it into the `files` directory of this repo. The download location will 
  change after AAP 2.1 is released properly.
- create a vault to store your credentials: `rh_username` and `rh_password` for your
    access.redhat.com credentials, and `aap_admin_pw`, `aap_db_pw`, `hub_admin_pw` and
    `hub_db_pw` for the application and database passwords. Easiest is to create it as
    `group_vars/all/vault.yml`.

Note: the default plan users the same storage pool ('default') for all 6 VMs. That is
a lot to stomach for most storage devices. If you have more than one disk, consider
creating a storage pool on it and spreading your VMs over multiple pools.


## Deploying the plan

### 

### What are the different available plans?
Because not everyone has a really beefy machine, I have created various plans to enable
as many people as possible to enjoy automated demo deployment:

- aap2_cluster.yml deploys a three node controller cluster with 8192MiB of RAM each,
  an HAProxy load balancer with 2048MiB of RAM, an Automation Hub machine with another
  8192MiB of RAM and a database - shared between the controller and Hub machines - with
  4096MiB of RAM
- aap2_cluster_small deploys two controller nodes only, and brings down the amount of
  required memory significantly, by using only 4096MiB of RAM for the controller nodes,
  and only 2048MiB for the database, load balancer and Hub machines. This also requires
  passing required_ram=1800 to the installer, resulting in a cluster that is unsupported
  (but will run a simple demo just fine in my experience).
- aap2_cluster_only does the same as aap2_cluster_small, but does not deploy the Hub
- aap2_single_node does the same as aap2_cluster_only, but with a single node controller
  "cluster"

### Actually running the plan
After filling in the pre-requisites - which you have to do only once per machine - you
can deploy the plan from the root of this project by running:
```
kcli create plan aap-cluster -f plans/aap2_cluster_full.yml
```

To prevent the machines from updating and installing git, mc and vim, run this instead:

```
kcli create plan aap-cluster -f plans/aap2_cluster_plans.yml -P skip_updates=true -P skip_utilities=true
```

Wait for a couple of minutes for your plan to spin up and you are done :)

#### DNS
If you want, you can drop this content into
`/etc/NetworkManager/dispatcher.d/pre-up.d/97-lab.sh` or something similar, and manually
execute it the first time the `redhat.lab` network is created (after a reboot, this'll
be automatic) to get DNS resolving for the `redhat.lab` domain:

```bash
#!/bin/sh
# This is a NetworkManager dispatcher script to configure split DNS for
# the 'crc' libvirt network.
# The corresponding crc bridge is recreated each time the system reboots, so
# it cannot be configured permanently through NetworkManager.
# Changing DNS settings with nmcli requires the connection to go down/up,
# so we directly make the change using resolvectl

export LC_ALL=C

if [ "$1" = lab ]; then
        resolvectl domain "$1" redhat.lab
        resolvectl dns "$1" 192.168.125.1
        resolvectl default-route "$1" false
        resolvectl dnssec "$1" no
fi

exit 0
```

## Cleaning up
As you are running RHEL8 images, when after running `kcli delete plan aap-cluster`,
you'll need to manually remove the subscription allocations on access.redhat.com.

## Pull requests, feature requests, etc.
Please send them :)

Some things tho: I know it is  possible to split the parameters for each plan into
a _default.yml file.  That, however, brings it's own problems with calling the ansible
playbook, the location of the vault and templates and other things. Therefore, I am
inclined not to split them for now.

