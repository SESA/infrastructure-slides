---
title: HaaS Bootstrap Demo
---
# SESA on the MOC

### node configuration and deployment

---

## Overview

- Default network boot configuration
- Configure and deploy applications

---

## Backgroud
- EbbRT OS for distributed Cloud applications
- Experimentation and benchmarking
    - networking latency/throughput
    - elastic deployment
    - native vs. virtualized
- HaaS bare-metal deployment

---

## Challenge
- Boot machine with particular configuration 
- Deploy and capture results 
- Identical redeployments

---

## Problem
- Many configurations
- Changes applied across configurations

---

## Early Solution
- Debirf: Diskless all-in-ram images
        - kernel + initfs
        - application + dependancies 
- State is hard-coded and contained
- One image, one configuration

---

## Problem
- images needs to be re-built on change 
- no transparecy into state of build image

---

## Current Solution
- `nfs`/`overlayfs` base system image
- SaltStack remote configuration and execution

---

## Network Boot (Headnode) 
- PXE / TFTP 
- Linux boots **read-only** `nfsroot`
        kernel fs/vmlinuz
        initrd fs/initrd.img
        append root=/dev/nfs nfsroot=$IP:/var/lib/tftpboot/fs ro       
- initrd uses `overlayfs` to mount R/W `tmpfs`

---

## SaltStack
#### Remote Configuration and Execution
- Salt Master on Headnode
- Nodes pre-configured as Salt Minions

---

