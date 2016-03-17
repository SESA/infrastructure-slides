---
title: SESA Architecture and Application Metastructure 
---

# Deployment of the EbbRT Application Model
**SESA Research Group**

Speaker: Jim Cadden 

---

### Scalable Elastic Systems Architecture
![](https://github.com/SESA/EbbRT/raw/master/doc/img/objective.png)
#### EbbRT Application Model

- App-specific OS, composed of reusable components 
- modular, light-weight, event-driven


==

### Scalable Elastic Systems Architecture
![](https://github.com/SESA/EbbRT/raw/master/doc/img/objective.png)

- App-specific OS, composed of reusable components 
- modular, light-weight, event-driven

==

### EbbRT Application Model
![EbbRT](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.002.jpeg)

- Linux 'front-end' application's computation to a back-end 
- Controller and back-end communicate across private internal network

---

### Fetal Reconstruction Application
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.fi.png)

####3-D reconstruction of volumetric MRI from data 'slices'

==

### Fetal Reconstruction Application
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.004.jpeg)

1. Slices read into applications 
2. Threads spawned to process registration & reconstruction

==

### Fetal Reconstruction Application
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.004.jpeg)

- Reconstruction process is CPU intensive  
- Work is done in parallel within application (alt. GPUs)
- Client information is sensitive

==


## Can performance improved by a distributed implementation?

---

## A 'Simple' Distributed Deployment
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.005.jpeg)

==

### A 'Simple' Distributed Deployment
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.005.jpeg)

- Master node processes requests to/from clients
- Work is distributed to a set of minion workers 

---

## EbbRT-fetalReconstruction 
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.007.jpeg)

- Job data is sent to a listening server 
- Each job spawns a controller process and worker nodes 

==

## EbbRT-fetalReconstruction 
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.007.jpeg)

- Controller distributes data 'slices' and tasks  
- Finished transformations collected by controller 

==

## EbbRT-fetalReconstruction 
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.007.jpeg)

- Back-end isolation at machine level 
- Partially-computed data sent across 'private' network

---

## Mapping the Model to Infrastructure
![EbbRT](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.002.jpeg)

==

### Mapping the Model to Infrastructure
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.002.jpeg)

- On-demand provisioning of logical machines
- Creation of isolated networks
- Within view of application

==

## Requirement of Our Environment

- Development / debugging
  - console access, gdb
- Deployment / reproducible experimental results
  - core pinning, perf, disable dynamic overclock

---

## Existing IaaS Providers 

  - Public cloud provides (OpenStack, AWS)  
  - Institutional systems (Emulab) 

==


### OpenStack Allocation
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.os1.png)

==

### OpenStack Allocation
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.os2.png)

==

## Public Cloud System
### OpenStack/AWS

- web portals & existing toolbase
- *But...*
- Limited configuration (no gdb, perf, core-pinning, multiqueue support)

---

## Local Environment

- Allocate virtual machines back-end nodes
- Configure logical networks (taps/bridges) 

---

## Local Network Configuration

- Create linked bridge and tap
- Assign subnet to bridge 
- DHCP server process listening on subnet

==

### A whole lot of this...

```
ip link add name **BRIDGE** type bridge || exit 1
ip link set **BRIDGE** up || exit 1
ip addr add **SUB_NET** dev **BRIDGE** || exit 1
ip tuntap add dev **TAP** mode tap multi_queue || exit 1
ip link set **TAP  master **BRIDGE** || exit 1
ip link set **TAP** up || exit 1

dnsmasq --listen-address=HOSTIP -z --dhcp-range=10.10.10.50,10.10.10.150,12h \
--log-facility=/tmp/dnsmasq.log
```

---

## Local Environment
### VM Allocation 

- Spawn a VM with custom system image
- Configure machine specs of VM (vcpus, ram, numa)
- Attach VM to tap
- Log IO to disk 

==


## Local Environment
### VM Allocation 

```
/usr/local/bin/qemu-system-x86_64 -cpu host -enable-kvm -serial stdio -m 2G -smp cpus=2 -numa node,cpus=0 -numa node,cpus=1 --netdev tap,id=vlan1,ifname=kh_41,script=no,downscript=no,vhost=on,queues=2 --device virtio-net-pci,mq=on,vectors=6,netdev=vlan1,mac=02:00:00:04:00:29 -pidfile /opt/khpy/khdb/Networks/119/41/pid -display none  -kernel /home/jmcadden/work/ebbrt-contrib/apps/helloworld/baremetal/build/Release/helloworld.elf32
```

---

## Local Provisioning Service 
### Kittyhawk project 

`$ ./kh`

```
usage: kh [-h] {alloc,clean,network,rmnode,rmnet} ...
```

==

### Node Allocation in Kittyhawk
```
usage: kh alloc [-h] [--ram num] [--cpu num] [--pin pin] [--numa num]
                [--cmd CMD] [--perf PERF] [--diskimg] [-g] [-s] [-n num] [-t]

Allocate node for a given network

positional arguments:
  network      Network id
  img          Path to kenel / disk image
  config       Path to configuration (optional)

optional arguments:
  -h, --help   show this help message and exit
  --ram num    Total RAM (GB) (default: None)
  --cpu num    Total CPUs (default: None)
  --pin pin    Core list for taskset -c (e.g., 1-2,11,12 (default: None)
  --numa num   NUMA nodes (resources devided evenly) (default: None)
  --cmd CMD    Append string to end of qemu command (default: None)
  --perf PERF  Enable kvm perf (default: )
  --diskimg    Load as disk image (ignores config) (default: None)
  -g           Enable gdb server (default: None)
  -s           Signal on termination (default: None)
  -n num       Number of nodes (default: 1)
  -t           Test run: setup and print boot command without executing
               (default: None)
```

==

### Node Allocation in Kittyhawk

` ./kh alloc 119 helloworld.elf32 `

```
41
02:00:00:04:00:29
tap: kh_41
/opt/khpy/khdb/Networks/119/41/stdout
/opt/khpy/khdb/Networks/119/41/stderr
/opt/khpy/khdb/Networks/119/41/finish
/opt/khpy/khdb/Networks/119/41/cmd
```

==

### Node Allocation in Kittyhawk

` ./kh alloc 119 helloworld.elf32 /dev/null`

```
/usr/local/bin/qemu-system-x86_64 -cpu host -enable-kvm -serial stdio -m 2G -smp cpus=2 -numa node,cpus=0 -numa node,cpus=1 --netdev tap,id=vlan1,ifname=kh_41,script=no,downscript=no,vhost=on,queues=2 --device virtio-net-pci,mq=on,vectors=6,netdev=vlan1,mac=02:00:00:04:00:29 -pidfile /opt/khpy/khdb/Networks/119/41/pid -display none  -kernel /home/jmcadden/work/ebbrt-contrib/apps/helloworld/baremetal/build/Release/helloworld.elf32
```

==

### Kittyhawk Summary

`$ ./kh clean`

```
All your networks and nodes have been removed.
```

- Simple service interfaces, central management
- Allocation latency in seconds 
- Flexibility at OS level

---

## On-Demand Hardware Provisioning 

- Direct access to provisioned hardware (bare-metal clouds)
- Private network configuration

---

### Hardware as a Service (HaaS)

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.008.jpeg)

- Headnode and physical nodes attached to a private vlan
- Physical nodes network boot (PXE / TFTP)

==

### Hardware as a Service (HaaS)

```
$ haas
Usage: /usr/bin/haas <command> <arguments...>
```

- Interact with service through master node
- Allocate physical node on vLAN

==

### Hardware as a Service (HaaS)

`$ haas show-node cisco-26`

```
{"name": "cisco-26", "nics": [{"label": "enp130s0f0", "macaddr": "90:e2:ba:87:69:08", "networks": {"vlan/native": "SESA"}}], "project": "SESA"}

```

---

## On-Demand Hardware Provisioning 

#### Challenges of our Model

- Boot machine with particular configuration 
- Identical redeployments

==


## On-Demand Hardware Provisioning 

#### Early Solution

- One image, one configuration
- Diskless all-in-ram system image
        - kernel + initfs
        - application + dependancies 
- Identical redeployments 

==

## On-Demand Hardware Provisioning 

#### Early Solution

- Many images, many configurations
- Change applied across configurations
- Manual boot configuration 


---

## On-Demand Hardware Provisioning 

#### Current Solution

- Linux boots **read-only** `nfsroot`
- initrd uses `overlayfs` to mount R/W `tmpfs`

```
        kernel fs/vmlinuz
        initrd fs/initrd.img
        append root=/dev/nfs nfsroot=$IP:/var/lib/tftpboot/fs ro       
```

==

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.008.jpeg)

- Linux boots **read-only** `nfsroot`
- initrd uses `overlayfs` to mount R/W `tmpfs`

```
        kernel fs/vmlinuz
        initrd fs/initrd.img
        append root=/dev/nfs nfsroot=$IP:/var/lib/tftpboot/fs ro       
```

---

### On-Demand Hardware Provisioning Summary

- Control of the full software stack
- True-scale distributed deployments 
- Boot latencies ~5 minutes 
- EbbRT uses para-virtualized device drivers

==

### On-Demand Hardware Provisioning Problems?

- Boot latencies ~5 minutes 
- EbbRT uses para-virtualized device drivers

---

### What's Next?

- A distributed (Kittyhawk-like) provisioning service
- Combine the fast-boot of VMs/containers on configured HaaS hardware
- Elastic resource pool provisioning 
  - Client acquire VMs, system service acquires hardware
- Extend private vLan with Q-in-Q tunneling 

---

# Thank You!
## This concludes my talk. I will happy answer any questions.

https://github.com/SESA/EbbRT

https://github.com/SESA/khpy

https://github.com/CCI-MOC/haas
