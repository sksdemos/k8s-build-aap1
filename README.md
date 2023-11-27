# Bootstrap a Kubernetes Cluster `(k8s-build)`

Playbooks to bootstrap a custom sized Kubernetes cluster with HA and addons 

 - Default cluster sizing is 1 master and 2 workers
 - Default VM resources (Memory 2 Gb, vCPU 2, Disk 10 Gb) per node. Highly customizable
 - Allows for custom sizing of Kubernetes cluster
 - K8s master(s) in odd number count 3 or greater will deploy HAproxy and keepalived for HA
 - Playbooks create a separate libvirt network and libvirt storage. These VMs will not interfere with existing VMs
 - Support for Optional add-ons (more to be added):
     - Install kubectl on host
     - Install Octant Dashboard
     - NFS Storage class


## Known Issues

Most of these problems were observed in the development phase of the playbooks and may have occured due to nested virtualization, resource limitation and repeatitive playbook runs that re-created the VM infrastructure with the cloud-init and libvirt_infra roles causing the underlying libvirt infrastructure to get stressed. I hope none of you will be seeing these issues in your own deployment :)

On rare occasions you may encounter these failures which appears as follows:

### 1) Long Wait for cloud-init to create custom VMs
While bootstraping the cluster the cloud-init VMs take a very long time to boot (Typically 3-4 minutes). This is a known issue reported with the cloud-init images and have been reported by a lot of community members. This will show up as a task **"Waiting for SSH to be available on all VMs (Be Patient)"** so the only thing to do is just wait.


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
This is a Kubernetes bootstraping project and requires playbooks to run in a sequence as set in the master playbook to bring up the cluster. Some playbooks when run independently will not run in an idempotent way since they use the command module to execute the kubeadm init command. If the k8s masters are up or partially setup then rerunning the playbook will cause a run-time error. Some playbooks when syntax checked independently will generate syntax errors as they use in-memory inventory and make use of magic variable groups with an index. These playbooks will work fine during an actual run.



Good Luck!

