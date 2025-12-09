# Prerequisites

## server specification
| Type | Number of servers | vCPU | RAM Memory | Storage (system) | Storage (data) | IAS |
| :- | :- | :- | :- | :- | :- | :- |
| Bastion | 2 | 8 vCPU | 32 GB | 700 GB | 0 GB | 2 |
| Master | 3 | 8 vCPU | 32 GB | 120 GB | 0 GB | 1 |
| Workers | 3 | 16 vCPU | 64 GB | 300 GB | 0 GB | 1 |
| Infra | 3 | 16 vCPU | 64 GB | 120 GB | 0 GB | 1 |
| Bootstrap | 1 | 8 vCPU | 32 GB | 120 GB | 0 GB | 1 |

> Guacamole VM to download `RHEL 8.7 OS`

!!! note "Bastion Hosts"
    There are 2 Bastion Hosts:

    - **Online Bastion** (`RHEL 8.7 OS`): to download necessary resources(Images, Executables, Installation programs)
    - **Offline Bastion**:
        - located in the **internal network**
        - **Online Bastion** transfer downloaded Resources to this **Offline Bastion**
        - ⚠️ this is where the OpenShift Installation happens!
<!-- 
        - HTTP server
        - load balancer
        - image registry
        - DNS
        - OCP deployment utilities  -->

## network
we use **static IPs** instead of **DHCP**, so for each machine, we need to define:

- IP
- DNS
- hostname


Network: External	

- Virtual IP address space
- Virtual IP address range
- Number of available IP addresses

Network: Cluster	

- IP address space: `192.168.252.0/24`
- IP address range
- Subnet prefix
- Number of available IP addresses
- Number of IP addresses per node
- Maximum number of nodes

Network: Service	

- IP address space
- IP address range
- Number of available IP addresses

### dns
- servers:

    |Type| FQDN hostname | IP Address|
    | :- | :- | :- |
    |Bastion<br/> - offline|`bastion.ocp4.xxx.com`|`192.168.252.X`|
    | - online|`bastiononline.ocp4.xxx.com`|`192.168.252.X`|
    |Bootstrap|`bootstrap.ocp4.xxx.com`|`192.168.252.X`|
    |Master|`master01.ocp4.xxx.com`|`192.168.252.X`|
    ||`master02.ocp4.xxx.com`|`192.168.252.X`|
    ||`master03.ocp4.xxx.com`|`192.168.252.X`|
    |Infra|`infra01.ocp4.xxx.com`|`192.168.252.X`|
    ||`infra02.ocp4.xxx.com`|`192.168.252.X`|
    ||`infra03.ocp4.xxx.com`|`192.168.252.X`|
    |Worker|`worker01.ocp4.xxx.com`|`192.168.252.X`|
    ||`worker02.ocp4.xxx.com`|`192.168.252.X`|
    ||`worker03.ocp4.xxx.com`|`192.168.252.X`|
- services:

    |Type| FQDN hostname | IP Address|
    | :- | :- | :- |
    | VIP Apps | `*.apps.ocp4.xxx.com` | `192.168.252.X` |
    | VIP API | `api.ocp4.xxx.com` | `192.168.252.X` |
    | VIP API-INT | `api-int.ocp4.xxx.com` | `192.168.252.X` |


### load balancers
configure the HAProxy load balancer within the "offline" Bastion node. 

!!! note
    services 的 endpoints -> servers 的 endpoints

- subnet name: `ocp4.xxx.com`
- subnet name: `ocp4.xxx.com`




|Frontend| Targets | Port|
| :- | :- | :- |
|VIP Apps<br/>`*.apps.ocp4.xxx.com`|- Infra nodes' FQDN hostname<br>- Worker nodes' FQDN hostname|80|
|VIP Apps<br/>`*.apps.ocp4.xxx.com`|- Infra nodes' FQDN hostname<br>- Worker nodes' FQDN hostname|443|
|VIP API<br/>`api.ocp4.xxx.com`|- Bootstrap node' FQDN hostname<br>- Master nodes' FQDN hostname|6443|
|VIP API-INT<br/>`api-int.ocp4.xxx.com`|- Bootstrap node' FQDN hostname<br>- Master nodes' FQDN hostname|22623|



# Installation

## 1. Download resources
Enter **VMware vSphere vCenter**. Download <u>RHEL 8.7 DVD</u> and upload to the **VMware vSphere vCenter**

## 2. Create Online Bastion 
the **online bastion*+ will be a VM `RHEL 8.7` with internet connection

- name: `bastiononline.ocp4.xxx.com` <- this is the **FQDN hostname**
- localhost login:
    - User: `root`
    - PW: `Passw0rd`
- open **Network Manager Tool UI**: 
    ```bash
    nmtui
    ```
- IPv4:
    - Addresses: 192.168.252.22/24
    - Gateway: 192.168.252.1
    - DNS servers: 192.168.252.1

## 3. Create Offline Bastion
the **offline bastion*+ will be a VM `RHEL 8.7` <u>without</u> internet connection

- IPv4:
    - Addresses: 192.168.252.23/24
    - DNS servers: 192.168.252.23
    - Routing:
        - Destination/Prefix: 192.168.0.0/16
        - Next Hop: 192.168.252.1
        - Metric: 90

!!! note "Verification"
    ```bash
    ping 192.168.252.1
    # this should work

    ping 8.8.8.8
    # this should NOT work
    ```

## 4. in Online Bastion
- disable SELINUX
- disable firewall
    ```bash
    systemctl stop firewalld
    systemctl disable firewalld
    ```
- configure the hostname
- install required RedHat packages
    ```bash
    # 在文件中配置 “RHEL的本地 DVD 仓库”
    vi /etc/yum.repos.d/rhel87dvd.repo

    # 列出启用的仓库
    yum repolist enabled
    ```
- 安装依赖包
    ```bash
    yum install -y  podman \
        jq openssl httpd-tools curl wget telnet nfs-utils \
        httpd.x86_64 \
        bind bind-utils rsync mkisofs
    ```
- 安装 ansible
    ```bash
    scp ansible-2.9.27-1.el8ap.noarch.rpm root@192.168.252.22:/root
    yum localinstall -y ansible-2.9.27-1.el8ap.noarch.rpm
    ```
- 创建空文件结构
    ```txt
    /root
    |-- registry
        |-- auth
        |-- certs
        |-- data
        `-- downloads
            |-- images
            |-- secrets
            `-- tools
    ```
- 安装 OpenShift utilities 
    ```bash
    # 下载安装 OC client (oc, kubectl)
    wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.16.9/openshift-client-linux-amd64-rhel8-4.16.9.tar.gz
    tar -xvf openshift-client-linux-amd64-rhel8-4.16.9.tar.gz
    cp /root/registry/downloads/tools/oc /usr/bin/oc

    # 下载安装 OC installer 
    wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.16.9/openshift-install-linux-4.16.9.tar.gz
    tar -xvf openshift-install-linux-4.16.9.tar.gz
    cp /root/registry/downloads/tools/openshift-install /usr/bin/openshift-install

    # 下载安装 mirror-registry
    wget https://mirror.openshift.com/pub/cgw/mirror-registry/latest/mirror-registry-amd64.tar.gz
    tar -xvf mirror-registry-amd64.tar.gz

    # 下载安装 oc-mirror 
    wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/4.16.19/oc-mirror.tar.gz
    tar xvfz oc-mirror.tar.gz
    chmod +x /root/registry/downloads/tools/oc-mirror
    cp /root/registry/downloads/tools/oc-mirror /usr/local/bin/
    oc-mirror help

    # 下载安装 butane （Ignition 配置文件生成器）
    curl https://mirror.openshift.com/pub/openshift-v4/clients/butane/latest/butane-amd64 --output butane
    
    # 下载安装 ISO Maker
    cd /root/
    mkdir Coreos-iso-maker
    cd Coreos-iso-maker/
    yum install -y git
    git clone https://github.com/chuckersjp/coreos-iso-maker
    ```
- disk partition
    ```bash
    fdisk /dev/sda
    ```

    ```txt
    Welcome to fdisk (util-linux 2.32.1).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.


    Command (m for help): n
    Partition number (4-128, default 4): 
    First sector (1468004352-2147483614, default 1468004352): 
    Last sector, +sectors or +size{K,M,G,T,P} (1468004352-2147483614, default 2147483614): 

    Created a new partition 4 of type 'Linux filesystem' and of size 324 GiB.

    Command (m for help): w
    The partition table has been altered.
    Syncing disks.
    ```

    Expand the logical volume that has the operating system boot:
    ```bash
    pvcreate /dev/sda4
    vgs

    vgextend rhel /dev/sda4
    lvs

    lvextend -l +100%FREE /dev/rhel/root
    lsblk -f

    xfs_growfs /dev/rhel/root
    df -h
    ```
- download CoreOS image
    ```bash
    cd /root/registry/downloads/images/
    
    wget https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.16/4.16.3/rhcos-4.16.3-x86_64-metal.x86_64.raw.gz
    wget https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.16/4.16.3/rhcos-4.16.3-x86_64-live.x86_64.iso
    ```
- Mirror images
    ```bash
    # Prepare pull secret from https://cloud.redhat.com/openshift/install/pull-secret

    cd /root/registry/downloads/secrets/
    vi pull-secret.txt
    cat ./pull-secret.txt | jq . > ./pull-secret.json

    mkdir /run/user/0/containers
    cp /root/registry/downloads/secrets/pull-secret.json /run/user/0/containers/auth.json
    ls -lart /run/user/0/containers/auth.json

    # Mirror images
    cd /root/registry/
    oc mirror init  > imageset-config.yaml
    
    # Add configurations
    vi imageset-config.yaml
    ```

    ```yaml
    # imageset-config.yaml
    kind: ImageSetConfiguration
    apiVersion: mirror.openshift.io/v1alpha2
    archiveSize: 4
    storageConfig:
    local:
        path: /root/registry/data
    mirror:
    platform:
        architectures:
        - "amd64"
        channels:
        - name: stable-4.16
        type: ocp
        minVersion: 4.16.9
        maxVersion: 4.16.9
        shortestPath: true
    operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.16
        packages:
        - name: openshift-pipelines-operator-rh
        defaultChannel: latest
        channels:
        - name: latest
    additionalImages:
    - name: registry.redhat.io/ubi8/ubi:latest
    helm: {}
  ```

  ```bash
  # Use the defined config file "imageset-config.yaml"
  oc mirror --config=./imageset-config.yaml file:///root/registry/data

  # Test
  oc-mirror list operators --catalog registry.redhat.io/redhat/redhat-operator-index:v4.16
  ```

## 5. in Offline Bastion
