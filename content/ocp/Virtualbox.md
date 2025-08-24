---
title: Virtual Box 
weight: 3
date: 2017-01-05

---
## How to increase disk size in virtualbox
- Stop the vm on virtualbox
- check the disk 
```t
VBoxManage list hdds
```
- Increase the size to 50 GB
```t
VBoxManage modifyhd "E:\VirtualBox VMs\crc_default_1725632457848_7611\Rocky-9-Vagrant-Vbox-9.4-20240509.0.x86_64.vmdk"  --resize 51200
```