

+++
title = "OpenShift on Baremetal"
weight = 4
+++


## Prerequisites

- **Follow the GitHub Repository**  
  [ocp4-metal-install ](https://github.com/ryanhay/ocp4-metal-install)

- **Create a RHEL 8 or 9 Virtual Machine on VMware Workstation**
  - Ensure the VM has **two network interfaces (NICs)** configured as **Bridge**:
    - **NIC 1**: Connected to the **public network** (Internet access)
    - **NIC 2**: Used for **internal network communication**
      - Assign a static IP (e.g., `192.168.22.1/24`)
      - Learn how to generate and manage MAC addresses

- **Understand Firewalld Configuration**
  - Learn about:
    - `public` and `private` zones
    - How to open ports and allow services

- **Install and Configure a DHCP Server**
  - Learn how to assign static IPs using MAC addresses
  - Test by confirming DHCP leases to other VMs

- **Set Up DNS (BIND/named) on RHEL 8**
  - Make necessary DNS configuration changes
  - Verify DNS resolution from other VMs

- **Learn HAProxy**
  - Understand its role in API load balancing and routing

- **Set Up an HTTP Server with a Custom Port**
  - Host sample content
  - Access the hosted content from other VMs

- **Configure NFS for Storage Class Usage**

- **Set System Timezone to UTC**
  ```sh
  timedatectl
  sudo timedatectl set-timezone UTC
  ```

- **Check Network Devices**
  ```sh
  nmcli device status
  nmcli connection show
  ```

- **Activate a Network Connection**
  ```sh
  nmcli connection up <connection-name>
  ```

- **Add a New Network Profile (If IP is not assigned)**
  ```sh
  nmcli con add type ethernet ifname ens160 con-name ens160
  ```

- **Bring the Connection Up**
  ```sh
  nmcli con up ens160
  ```

## Configure Local YUM Repository Using ISO

1. **Mount the ISO Image**
   ```sh
   mount /dev/sr0 /mnt/
   ```

2. **Create a Local YUM Repository File**
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
