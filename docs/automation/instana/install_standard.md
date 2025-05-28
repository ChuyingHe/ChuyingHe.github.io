# Instana Standard Installation - Single Node

<img src="../imgs/instana_editions.png" width="500" />

## 1. Create a VM on fyre
Create a machine on fyre with at least 16Core, 64 Memory CoreOS. 

Using the <username>:<api_key> and the [fyre API](https://fyre.ibm.com/help#fyre-api) request to create the machine. Use `ssh` to access the VM's console

## 2. Preparing for a single-node deployment
> Doc: https://www.ibm.com/docs/en/instana-observability/current?topic=cluster-preparing

- Follow the instructions until th "**Create the directories**"
- Step **Add mount paths** ensures the device names remain the same after reboot. Skipping it for now
- `<base_domain>` is set to the fyre VM's public URL --> this can be changed in the CRD also

TO CHECK:
- Firewall rules
- Configuring an HTTP proxy

## 3. Installing
> Doc: https://www.ibm.com/docs/en/instana-observability/current?topic=edition-installing

In this step, we basically use the command `stanctl up` to automatically set up everything. Required parameters:

- download key or an official agent key: from Instana
- sales key: from instana
- domain: fyre VM's public URL
- tenant: tenant0
- unit: unit0
- admin password: setup yourself
- TLS: auto generate

> Tipp: 
> 
> - use `stanctl --help`
> - all configs are in the local directory `.stanctl/`


## Namespaces
### instana-core
all backend components, get the components' logs one by one to check potential
### instana-unit
web & tenant. The `username` & `pw` to login to the Instana UI are stored in a secret `instana-unit` in this NS. Decode the secret to get the credential:

```bash
echo xxx | base64 -d
```

# Install Instana Agent 
One Agent for one Node!

<img src="../imgs/agent.png" />


# Deinstallation
https://www.ibm.com/docs/de/instana-observability/287?topic=edition-uninstalling





# Instana Standard Installation - Multi Nodes - Airgap


[Doc](https://www.ibm.com/docs/en/instana-observability/287?topic=mnc-system-requirements)

||instana0|instana1|instana2|
|:-|:-|:-|:-|
|**Role**|backend|datastore|other|
|**Function**|- running backend workloads that require persistent volumes.<br/>- runs gateways, acceptors and UI backend|- running data stores|- running the rest of Instana workloads.|
|**Storage requirements**|Disks for directory `/objects`|Disks for directory `/data`(500GB), `/metrics`(1000GB), `/analytics`(1200GB)|-|
|**Private IP in LAN**|`10.0.0.0`|`10.0.0.1`|`10.0.0.2`|

!!! note "Tools"
    - check cpu: `lscpu | grep 'CPU(s)'`
    - check disk `lsblk`


!!! note "Airgap Modes"

    1. Airgap without mirror repo: then download all the images that related to instana
    2. Airgap with docker repo

## Bastion host
This bastion host should have internet connection and is responsible for downloading the stanctl and the related packages.
```bash
# download stanctl rpm
export DOWNLOAD_KEY=<instana-download-key>
export INSTANA_VERSION=1.8.1-1
wget https://_:$DOWNLOAD_KEY@artifact-public.instana.io/artifactory/rel-rpm-public-virtual/stanctl-$INSTANA_VERSION.x86_64.rpm
sudo rpm -Uvh stanctl-$INSTANA_VERSION.x86_64.rpm

# Verify the stanctl version
stanctl --version

# Check the available Instana version using the downloaded stanctl tool
stanctl versions identify

# Packing stanctl for air-gap environment, this might take a while
stanctl air-gapped package

```
Now I got a `instana-airgapped.tar.gz`, I copy it to `node0` using ssh:
```bash
scp instana-airgapped.tar.gz root@node0:/root/instana-airgapped.tar.gz
```

## node0

!!! danger "storage requirement"
    |Description|Default directory|Min size|Disk I use here|
    |:-|:-|:-|:-|
    |Object directory	|`/mnt/instana/stanctl/objects`|1000|`sdb`|
    	


### 0. Make Node offline
to simulate airgap environment, I simply sabotage the gateway IP by
```bash
# 1. check the gateway IP
ip route show
# default via 10.0.0.254 # <-- this is the gateway IP

# 2. Change the Gateway
sudo ip route replace default via <new-gateway-ip>
sudo ip route replace default via 10.0.0.250

```

### 1. Configuration

```bash
# 1. Make file system
for disk in sdb; do
    echo "make filesystem for $disk"
    mkfs.xfs -f -i size=1024 -L $disk /dev/$disk
done


# 2. Create the directories
mkdir -p /mnt/instana/stanctl/objects


# 3. Add mount path
# Backup fstab file
cp /etc/fstab /etc/fstab.backup
# Find out disk uuid
blkid
# Append changes to `fstab`
echo "UUID=<UUID-DISK-sdb>  /mnt/instana/stanctl/objects    xfs   discard,defaults,nofail    0 0" >> /etc/fstab
mount -a
systemctl daemon-reload
# Check result
lsblk


# 4. Configure kernel parameters
sh -c 'echo vm.swappiness=0 >> /etc/sysctl.d/99-stanctl.conf' && sysctl -p /etc/sysctl.d/99-stanctl.conf
sh -c 'echo fs.inotify.max_user_instances=8192 >> /etc/sysctl.d/99-stanctl.conf' && sysctl -p /etc/sysctl.d/99-stanctl.conf
grubby --args="transparent_hugepage=never" --update-kernel ALL
# A system reboot is required for these changes to take effect.
reboot
# verfiy the change, you should see: always madvise [never]
cat /sys/kernel/mm/transparent_hugepage/enabled


# 5. Edit $PATH
# echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
# source ~/.bashrc
# echo $PATH



# 6. Open ports with rules for Firewall
systemctl status firewalld
firewall-cmd --permanent --add-port=22/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=8443/tcp
firewall-cmd --new-zone=internal-access --permanent
# ⚠️⚠️⚠️ Replace the node IPs here!
firewall-cmd --permanent --zone=internal-access --add-source=<node0-IP>	
firewall-cmd --permanent --zone=internal-access --add-source=<node1-IP>
firewall-cmd --permanent --zone=internal-access --add-source=<node2-IP>
firewall-cmd --permanent --zone=internal-access --add-port=22/tcp
firewall-cmd --permanent --zone=internal-access --add-port=6443/tcp
firewall-cmd --permanent --zone=internal-access --add-port=10250/tcp
firewall-cmd --permanent --zone=internal-access --add-port=2379/tcp
firewall-cmd --permanent --zone=internal-access --add-port=2380/tcp
firewall-cmd --permanent --zone=internal-access --add-port=5001/tcp
firewall-cmd --permanent --zone=internal-access --add-port=8472/udp
firewall-cmd --permanent --zone=internal-access --add-port=9443/tcp
firewall-cmd --permanent --zone=internal-access --add-port=53/udp
firewall-cmd --permanent --zone=internal-access --add-port=53/tcp
firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16
firewall-cmd --permanent --zone=trusted --add-source=10.43.0.0/16
firewall-cmd --permanent --zone=trusted --add-interface=lo
firewall-cmd --reload
```

### 2. SSH configuration
**SSH configuration** is required only in a multi-node cluster. The `node0` must be able to access `node1` and `node2` by using SSH.

```bash
# generate ssh key pairs
ssh-keygen -t rsa

# output the Public Key 
# copy the output to node1 and node2
# cat /root/.ssh/id_rsa.pub

# Copies your public SSH key to a remote host's authorized_keys file (/root/.ssh/authorized_keys)
ssh-copy-id -i /root/.ssh/id_rsa root@instana-1
ssh-copy-id -i /root/.ssh/id_rsa root@instana-2
# Prompt will remind you to type in the ssh password to both hosts (instana-1 and instana-2)
```

### 3. stanctl
In a multi-node cluster, you must install stanctl on `node0`. I got `instana-airgapped.tar.gz` from a connected environment - in our case the `student` machine.

```bash
# 1. Extracting archives to the target directory '/usr/local/bin'
tar -xzf instana-airgapped.tar.gz -C /usr/local/bin --strip-components 1 airgapped/stanctl


# 2. Extract air-gapped instana files: no mirror-repositories
stanctl air-gapped import --file  instana-airgapped.tar.gz

# 3. creating instana
stanctl up --air-gapped --multi-node-enable
# ? Select installation type for multi-node cluster installation: production
# ? Enter the IP addresses for all three nodes, separated by commas. The first IP is for the backend, the second is for the datastore (e.g., 10.8.1.10,10.8.1.11,10.8.1.12): 10.0.0.30,10.0.0.31,10.0.0.32
# ? Enter the download key or an official agent key: **********************
# ? Enter the sales key: **********************
# ? Enter the domain under which Instana will be reachable: prod.instana.training
# ? Enter your tenant name: labs
# ? Enter your unit name: student
# ? Enter Instana admin password: ********
# ? Confirm Instana admin password: ********
# ? Enter TLS certificate file (hit ENTER to auto-generate):
```


!!! note "domain"
    the domain can be found in in the file `/etc/hosts`. The document contains content in the following format:
    ```bash
    <IP Address> <Hostname> [Alias...]
    ```
    I saw:
    ```bash
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    10.0.0.30   instana-0 ... student-labs.prod.instana.training ...
    ```

    Seeing `student-labs.prod.instana.training`, we make:

    - base domain: `prod.instana.training`
    - unit: `student`
    - tenent: `labs`

!!! note "UI URL of Instana Platform"
    In this case the Instana URL is `student-labs.prod.instana.training`

### Install agent
install agent, it will create pods for each nodes.


```bash
stanctl agent apply
```

## node1
!!! danger "storage requirement"
    |Description|Default directory|Min size|Disk I use here|
    |:-|:-|:-|:-|
    |Data directory	|`/mnt/instana/stanctl/data`|500|`sdb`|
    |Metrics directory	|`/mnt/instana/stanctl/metrics`|1000|`sdc`|
    |Analytics directory	|`/mnt/instana/stanctl/analytics`|1200|`sdd`|
    


### 1. Configuration
```bash
# 1. Make file system
for disk in sdb sdc sdd; do
    echo "make filesystem for $disk"
    mkfs.xfs -f -i size=1024 -L $disk /dev/$disk
done


# 2. Create the directories
mkdir -p /mnt/instana/stanctl/data
mkdir -p /mnt/instana/stanctl/metrics
mkdir -p /mnt/instana/stanctl/analytics


# 3. Add mount path
# Backup fstab file
cp /etc/fstab /etc/fstab.backup
# Find out disk uuid
blkid
# Append changes to `fstab`
echo "UUID=<UUID-DISK-500GB>   /mnt/instana/stanctl/data       xfs   discard,defaults,nofail    0 0" >> /etc/fstab
echo "UUID=<UUID-DISK-1000GB>  /mnt/instana/stanctl/metrics    xfs   discard,defaults,nofail    0 0" >> /etc/fstab
echo "UUID=<UUID-DISK-1200GB>  /mnt/instana/stanctl/analytics  xfs   discard,defaults,nofail    0 0" >> /etc/fstab
mount -a
systemctl daemon-reload
# Check result
lsblk


# 4. Configure kernel parameters
sh -c 'echo vm.swappiness=0 >> /etc/sysctl.d/99-stanctl.conf' && sysctl -p /etc/sysctl.d/99-stanctl.conf
sh -c 'echo fs.inotify.max_user_instances=8192 >> /etc/sysctl.d/99-stanctl.conf' && sysctl -p /etc/sysctl.d/99-stanctl.conf
grubby --args="transparent_hugepage=never" --update-kernel ALL
# A system reboot is required for these changes to take effect.
reboot
# verfiy the change, you should see: always madvise [never]
cat /sys/kernel/mm/transparent_hugepage/enabled

# 5. Edit $PATH
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
echo $PATH


# 6. Open ports with rules for Firewall
systemctl status firewalld
firewall-cmd --permanent --add-port=22/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=8443/tcp
firewall-cmd --new-zone=internal-access --permanent
# ⚠️⚠️⚠️ Replace the node IPs here!
firewall-cmd --permanent --zone=internal-access --add-source=<node0-IP>	
firewall-cmd --permanent --zone=internal-access --add-source=<node1-IP>
firewall-cmd --permanent --zone=internal-access --add-source=<node2-IP>
firewall-cmd --permanent --zone=internal-access --add-port=22/tcp
firewall-cmd --permanent --zone=internal-access --add-port=6443/tcp
firewall-cmd --permanent --zone=internal-access --add-port=10250/tcp
firewall-cmd --permanent --zone=internal-access --add-port=2379/tcp
firewall-cmd --permanent --zone=internal-access --add-port=2380/tcp
firewall-cmd --permanent --zone=internal-access --add-port=5001/tcp
firewall-cmd --permanent --zone=internal-access --add-port=8472/udp
firewall-cmd --permanent --zone=internal-access --add-port=9443/tcp
firewall-cmd --permanent --zone=internal-access --add-port=53/udp
firewall-cmd --permanent --zone=internal-access --add-port=53/tcp
firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16
firewall-cmd --permanent --zone=trusted --add-source=10.43.0.0/16
firewall-cmd --permanent --zone=trusted --add-interface=lo
firewall-cmd --reload

```

### 2. SSH configuration

```bash
# Copy the public key content to the $HOME/.ssh/authorized_keys
echo "<Public-key>" >> /root/.ssh/authorized_keys
```


## node3

### 1. Configuration

```bash
# 1. Configure kernel parameters
sh -c 'echo vm.swappiness=0 >> /etc/sysctl.d/99-stanctl.conf' && sysctl -p /etc/sysctl.d/99-stanctl.conf
sh -c 'echo fs.inotify.max_user_instances=8192 >> /etc/sysctl.d/99-stanctl.conf' && sysctl -p /etc/sysctl.d/99-stanctl.conf
grubby --args="transparent_hugepage=never" --update-kernel ALL
# A system reboot is required for these changes to take effect.
reboot
# verfiy the change, you should see: always madvise [never]
cat /sys/kernel/mm/transparent_hugepage/enabled


# 2. Edit $PATH
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
echo $PATH


# 3. Open ports with rules for Firewall
systemctl status firewalld
firewall-cmd --permanent --add-port=22/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=8443/tcp
firewall-cmd --new-zone=internal-access --permanent
# ⚠️⚠️⚠️ Replace the node IPs here!
firewall-cmd --permanent --zone=internal-access --add-source=<node0-IP>	
firewall-cmd --permanent --zone=internal-access --add-source=<node1-IP>
firewall-cmd --permanent --zone=internal-access --add-source=<node2-IP>
firewall-cmd --permanent --zone=internal-access --add-port=22/tcp
firewall-cmd --permanent --zone=internal-access --add-port=6443/tcp
firewall-cmd --permanent --zone=internal-access --add-port=10250/tcp
firewall-cmd --permanent --zone=internal-access --add-port=2379/tcp
firewall-cmd --permanent --zone=internal-access --add-port=2380/tcp
firewall-cmd --permanent --zone=internal-access --add-port=5001/tcp
firewall-cmd --permanent --zone=internal-access --add-port=8472/udp
firewall-cmd --permanent --zone=internal-access --add-port=9443/tcp
firewall-cmd --permanent --zone=internal-access --add-port=53/udp
firewall-cmd --permanent --zone=internal-access --add-port=53/tcp
firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16
firewall-cmd --permanent --zone=trusted --add-source=10.43.0.0/16
firewall-cmd --permanent --zone=trusted --add-interface=lo
firewall-cmd --reload
```

### 2. SSH configuration

```bash
# Copy the public key content to the $HOME/.ssh/authorized_keys
echo "<Public-key>" >> /root/.ssh/authorized_keys
```

## result

you can access the UI from Bastion workstation.




# Bug Documentation

!!! danger "error"
    **Error**: 

    ```bash
    running "ssh [-o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -o NumberOfPasswordPrompts=0 -o LogLevel=ERROR root@10.0.0.31 systemctl start k3s]" failed with exit code 1 (Job for k3s.service failed because the control process exited with error code.
    See "systemctl status k3s.service" and "journalctl -xeu k3s.service" for details.
    ```

    **Reason**: firewall setting is wrong, you might forgot to add the followings, check the firewall setting in this page:

    **Solution**:
    ```bash
    firewall-cmd --permanent --zone=internal-access --add-source=<node0-IP> ...
    firewall-cmd --reload
    ```

!!! danger "error"
    **Error**: 

    ```bash
    Executable k3s binary not found at /usr/local/bin/k3s

    # or
    Error: stat /root/.stanctl/k3s/k3s: no such file or directory
    ```

    **Reason**: k3s installation went wrong

    **Solution**: delete the cluster, clean up the installed stanctl and redo the
    ```bash
    # remove k3s
    source /usr/local/bin/k3s-uninstall.sh
    rm -rf /etc/rancher/k3s
    rm -rf /var/lib/rancher/k3s
    rm -rf /var/lib/kubelet
    rm -rf /etc/cni
    rm -rf /opt/cni
    rm -rf /var/lib/cni
    rm -rf ~/.kube 


    # remove stanctl
    rm -rf /root/.stanctl
    rm -f /usr/local/bin/stanctl
    ```

    then re-do the [STANCTL](#3-stanctl) step