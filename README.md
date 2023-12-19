# Bootstrap a Kubernetes Cluster `(k8s-build)`

Playbooks to bootstrap a custom sized Kubernetes cluster with HA and addons 

 - Default cluster sizing is 1 master and 2 workers. Sizing is customizable.
 - Default VM resources (Memory 2 Gb, vCPU 2, Disk 20 Gb thin provisioned) per node. Highly customizable
 - Allows for custom sizing of Kubernetes cluster
 - K8s master(s) in odd number count 3 or greater will deploy HAproxy and keepalived for HA
 - Playbooks create a separate libvirt network and libvirt storage. These VMs will not interfere with existing VMs
 - Support for Optional add-ons (more to be added):
     - Install kubectl on host
     - Install Octant Dashboard
     - NFS Storage class


## Testing Environment
     - Fedora 37 as host
     - Cloud images used: Centos 8 Stream and Centos 9 Stream
     - 300 Mbps internet connection

## Prerequisites

- Ansible 2.9
- KVM with Libvirt (Automatically configured by k8s-build)
- Hardware Virtualization to be enabled
- High speed internet connectivity
- Linux host that runs Fedora 37 or a Centos 8 or Centos 9 stream with about 8Gb of spare memory and 200Gb of disk space.
- Linux host can be a Virtual Machine with nested virtualization with 8Gb of memory and 200Gb of disk space preferably thin provisioned. 
- Linux host requires internet access
- Login account on Linux host with sudo privileges with a sudo password.


## How to setup the cluster?

The k8s-build project uses a set of playbooks that will be run in a sequence to stand up the Kubernetes Cluster in 7 - 15 mins depending on the speed of your hardware. On launch of the master playbook, a libvirt network (k8s-net) and a libvirt storage (k8s-store) are created. This ensures that this build will not interfere with existing virtual machines that you may already have. The libvirt network uses a class C network of 192.168.200.0 where gateway for all VM created is 192.168.200.1 and DHCP range is between 192.168.200.151 and 192.168.200.254. Of'course the network and the DHCP range is customizable. The pool of static ip addresses range is available from 192.168.200.1 - 192.168.200.150. From this pool the following static ip addresses are consumed. The ip address 192.168.200.2 is used as a virtual IP by keepalived for HA cluster. The ip address range 192.168.200.3 - 192.168.200.6 is consumed by the cluster VMs and a utility VM as per default cluster size of 1 master and 2 workers and a utility machine. This range will extend depending on how the cluster is size is further customized by you.

**Assumption:** Lets assume that the login user id is: **james** and his password is **GobbleDeGook**

### Installing Ansible on host machine
--- Install pip
$ sudo dnf install python3-pip

--- Upgrade pip
$ pip3 install --upgrade pip

--- Install Ansible 2.9 in the logged in user account
$ pip install --user 'ansible<2.10'

--- Clone the k8s-build project from the github repository
$ git clone https://github.com/sksdenos/k8s-build



### Cluster Sizing and customization
Cluster sizing and customization can be achieved by making minor changes to some of the settings files.




## Known Issues

Most of these problems were observed in the development phase of the playbooks and may have occurred due to nested virtualization, resource limitation and repetitive playbook runs that re-created the VM infrastructure with the cloud-init and libvirt_infra roles causing the underlying libvirt infrastructure to get stressed. I hope none of you will be seeing these issues in your own deployment :)

On rare occasions you may encounter these failures which appears as follows:

### 1) Long Wait for cloud-init to create custom VMs
While bootstrapping the cluster the cloud-init VMs take a very long time to boot (Typically 3-4 minutes). This is a known issue reported with the cloud-init images and have been reported by a lot of community members. This will show up as a task **"Waiting for SSH to be available on all VMs (Be Patient)"** so the only thing to do is just wait.


### 2) Bridge Failures
- fatal: [localhost]: FAILED! => {"changed": false, "msg": "error creating bridge interface br0-k8s: File exists"}
- fatal: [localhost]: FAILED! => {"changed": false, "msg": "error creating bridge interface virbr0: File exists"}

The above is an issue with libvirt that has occured very rarely and will go away when you delete the bridge interface and re-run the master playbook. I have not been able to understand the circumstance to replicate the issue. It just happens when it happens. To delete the bridge run the following command:

**# nmcli device delete virbr0**
**# nmcli device delete br0-k8s**


### 3) Strange Privilege Escalation failures
- fatal: [192.168.200.12]: FAILED! => {"changed": false, "checksum": "34e17debf4a4a1b5743871ab975af918135df3bd", "msg": "Destination /etc/modules-load.d not writable"}

The above issue happens very occasionally and I have not been able to replicate the issue to understand it better. Just re-run the master playbook again and this error dissappears.


### 4) Idempotency and Syntax errors 
This is a Kubernetes bootstraping project and requires playbooks to run in a sequence as set in the master playbook to bring up the cluster. Some playbooks when run independently will not run in an idempotent way since they use the command module to execute the kubeadm init command. If the k8s masters are up or partially setup then rerunning the playbook will cause a run-time error. Some playbooks when syntax checked independently will generate syntax errors as they use in-memory inventory and make use of magic variable groups with an index pointing to a specific position of the host list that can be only available during runtime. These playbooks will work fine during an actual run.


### 5) Centos 9 cloud images
Using Centos 9 stream cloud images for building VMs will throw deprication warning messages indicating that platform python has to be used but ansible 2.9 is using /usr/bin/python. For these deprecation_warnings=false has been set in ansible.cfg



Good Luck!

