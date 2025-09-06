+++
title = "OCP: Day 01"
weight = 1
duration = "1.5h"
+++

## Prerequisites
- [Follow Github Repo](https://github.com/ryanhay/ocp4-metal-install)

- Change timezone to UTC
```sh
timedatectl
sudo timedatectl set-timezone UTC
```
- check network device status
```sh
nmcli device status
nmcli connection show
```
- Activate
```sh
nmcli connection up <connection name>
```
- Add Network profile if ip address is not taken
```sh
nmcli con add type ethernet ifname ens160 con-name ens160
```
- Bring the connection up
```sh
nmcli con up ens160
```

## Configure Local yum using ISO
- Mount Iso Image
```sh
mount /dev/sr0 /mnt/
```
- Create the yum repo 
```sh
cat << EOF > /etc/yum.repos.d/rhel8-local.repo
[BaseOS]
name=RHEL8 BaseOS
baseurl=file:///mnt/BaseOS
enabled=1
gpgcheck=0


[AppStream]
name=RHEL8 AppStream
baseurl=file:///mnt/AppStream
enabled=1
gpgcheck=0

EOF
```