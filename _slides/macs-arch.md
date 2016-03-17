---
title: SESA Architecture and Application Metastructure 
---

## Scalable Elastic Systems Architecture
## Deployment Infrastructure and Application Model
### SESA Research Group

Speaker: Jim Cadden

---

## Scalable Elastic Systems Architecture
### EbbRT Application Model

![](https://github.com/SESA/EbbRT/raw/master/doc/img/objective.png)
- Back-end OS customized to the application (e.g, .. )

==

![EbbRT](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.002.jpeg)

- Linux front-end linked with EbbRT library
- Offload application's computation to a back-end 
- Controller and back-end communicate across private internal network

---

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.003.jpeg)
## Fetal Reconstruction Application
3-D reconstruction of volumetric MRI from data 'slices'

==

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.004.jpeg)

1. slices read into applications 
2. threads spawned to process registration & reconstruction
3. reconstructed slices rebuilt into rendering 

==

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.004.jpeg)

Considerations?
- Sensitive information of client(s)
- Isolation provided by OS

==

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.004.jpeg)

Problems?
- Reconstruction throughput is limited by the machine capacities
- Work is done in parallel internal to the application 


==

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.005.jpeg)

### Can we improve application throughput through a distributed implementation?

---

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.006.jpeg)
## A 'Typical' Distributed Deployment
1. slices read into applications 
2. threads spawned to process registration & reconstruction
3. reconstructed slices rebuilt into rendering 

==

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.006.jpeg)

Problems? 
- Partially transformed data sent across wire
- Different client data shared between workers

---

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.009.jpeg)
## EbbRT-fetalReconstruction 
- Job data is sent to a listening server 
- Each job spawns a controller process and multiple worker nodes 

==

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.007.jpeg)
## EbbRT-fetalReconstruction 
- Controller distributes data 'slices' and tasks  
- Finished transformations collected by controller and returned

==

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.007.jpeg)
## EbbRT-fetalReconstruction 
- Back-end isolation at machine level 
- Partially-computed data sent across 'private' network

---

![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.002.jpeg)
## Mapping the Model to Infrastructure
- on-demand provisioning of machines
- creation of isolated networks

---

## Local Environment
- allocate virtual machines back-end nodes
- configure logical networks (taps/bridges) 

==

## Local Environment
### Network Configuration (Linux)

- Create linked bridge and tap
- Assign subnet to bridge 
- DHCP server process listening on subnet

==

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
- Spawn a VMs with custom system image
- Configure machine specs of VM 
- Attach VM to tap 
- Log IO to disk 

==


## Local Environment
### VM Allocation 
```
/usr/local/bin/qemu-system-x86_64 -cpu host -enable-kvm -serial stdio -m 2G -smp cpus=2 -numa node,cpus=0 -numa node,cpus=1 --netdev tap,id=vlan1,ifname=kh_41,script=no,downscript=no,vhost=on,queues=2 --device virtio-net-pci,mq=on,vectors=6,netdev=vlan1,mac=02:00:00:04:00:29 -pidfile /opt/khpy/khdb/Networks/119/41/pid -display none  -kernel /home/jmcadden/work/ebbrt-contrib/apps/helloworld/baremetal/build/Release/helloworld.elf32
```

---

## Local Environment
### Kittyhawk Provisioning Service 

```
$ ./kh
usage: kh [-h] {alloc,clean,network,rmnode,rmnet} ...
```

[https://github.com/SESA/EbbRT](https://github.com/SESA/EbbRT)

---

### Kittyhawk Node Allocation
```
usage: kh alloc [-h] [--ram num] [--cpu num] [--pin pin] [--numa num]
                [--cmd CMD] [--perf PERF] [--diskimg] [-g] [-s] [-n num] [-t]
                network img config

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

### Kittyhawk Node Allocation
```
kh alloc 119 helloworld.elf32 /dev/null
41
02:00:00:04:00:29
tap: kh_41
/opt/khpy/khdb/Networks/119/41/stdout
/opt/khpy/khdb/Networks/119/41/stderr
/opt/khpy/khdb/Networks/119/41/finish
/opt/khpy/khdb/Networks/119/41/cmd
```

==

### Kittyhawk Node Allocation
```
/usr/local/bin/qemu-system-x86_64 -cpu host -enable-kvm -serial stdio -m 2G -smp cpus=2 -numa node,cpus=0 -numa node,cpus=1 --netdev tap,id=vlan1,ifname=kh_41,script=no,downscript=no,vhost=on,queues=2 --device virtio-net-pci,mq=on,vectors=6,netdev=vlan1,mac=02:00:00:04:00:29 -pidfile /opt/khpy/khdb/Networks/119/41/pid -display none  -kernel /home/jmcadden/work/ebbrt-contrib/apps/helloworld/baremetal/build/Release/helloworld.elf32
```

---

### Kittyhawk Summary
```
$ ./kh clean
All your networks and nodes have been removed.
```

## Public Cloud System
### OpenStack 
- deploy VMs through client interfaces

==

### OpenStack Allocation
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.os1.jpeg)

==

### OpenStack Allocation
![](https://dl.dropboxusercontent.com/u/11161963/mactalk/mactalk.os2.jpeg)

==

## Public Cloud System
### OpenStack 
- web portals & existing tool base
#### But...
- Limited configuration (no gdb, perf, core-pinning, multiqueue support)

---

## On-Demand Hardware Provisioning 
- Allocate bare-metal nodes
- Isolated by vlans

==

## On-Demand Hardware Provisioning 
### Hardware as a Service (HaaS)

==

### Hardware as a Service (HaaS)
- Interact with service via master node
- Allocate node on vlan
```
$ haas
Usage: /usr/bin/haas <command> <arguments...>
```
==

### Hardware as a Service (HaaS)
- Allocate node on project vlan
```
$ haas show_node cisco-26
{"name": "cisco-26", "nics": [{"label": "enp130s0f0", "macaddr": "90:e2:ba:87:69:08", "networks": {"vlan/native": "SESA"}}], "project": "SESA"}

```

==

### Hardware as a Service (HaaS)
## Project Headnode

## Network Boot (Headnode) 
- PXE / TFTP 
- Linux boots **read-only** `nfsroot`
        kernel fs/vmlinuz
        initrd fs/initrd.img
        append root=/dev/nfs nfsroot=$IP:/var/lib/tftpboot/fs ro       
- initrd uses `overlayfs` to mount R/W `tmpfs`





---

