# Prerequisites

## server specification
| Type | Number of servers | vCPU | RAM Memory | Storage (system) |
| :- | :- | :- | :- | :- |
| Bastion | 1 | 8 vCPU | 32 GB | 700 GB |
| Master | 3 | 8 vCPU | 16 GB | 120 GB |
| Workers | 3 | 16 vCPU | 32 GB | 300 GB |
| Infra | 3 | 8 vCPU | 16 GB | 120 GB |
| Bootstrap | 1 | 8 vCPU | 16 GB | 120 GB |

## vsphere
- vsphere_username: user@vsphere.local
- vsphere_password: xxx
- vsphere_hostname: ocpgym-vc.techzone.MyCloud.local
- vsphere_datastore: xxxYYY-storage
- vsphere_cluster: my-ocp-cluster
- vsphere_network: xxxYYY-segment
- vsphere_datacenter: MyCloud
- vsphere_folder: /MyCloud/vm/my-ocp-cluster/reservations/xxxYYY
- vsphere_resource_pool: /MyCloud/host/my-ocp-cluster/Resources/Cluster Resource Pool/Gym Member Resource Pool/xxxYYY
- vsphere_api_vip: 192.168.252.3
- vsphere_ingress_vip: 192.168.252.4
- base_domain: my-ocp-cluster.lan
- cluster_name: ocpinstall

## network
	
Network: External	

- Virtual IP API address
- Virtual IP Ingress address
- Virtual IP address range
- Number of available IP addresses

Network: Cluster	

- IP address space
- IP address range
- Subnet prefix
- Number of available IP addresses
- Number of IP addresses per node
- Maximum number of nodes

Network: Service	

- IP address space
- IP address range
- Number of available IP addresses

# Overview
Machines we will have:

- Bastion: scripts, files 等工具
- Bootstrap
- Master Nodes
- Worker Nodes
- [Optional] Infra Nodes

# Installation

## 1. SSH
```bash
# 1. 生成一个 ED25519 算法的 SSH 密钥对: ocpgym 和 ocpgym.pub
ssh-keygen -t ed25519 -N '' -f ~/.ssh/ocpgym

# 2. 启动 “SSH 代理” & 设置环境变量
eval "$(ssh-agent -s)"

# 3. 将私钥（ocpgym文件）交给 “SSH 代理”
ssh-add ~/.ssh/ocpgym
```

## 2. Binaries

```bash
mkdir ocpinstall
cd ocpinstall

# Download binaries
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz

# Extract to get binaries
tar -xvf openshift-client-linux.tar.gz      # oc, kubectl
tar -xvf openshift-install-linux.tar.gz     # openshift-install

# Add the binaries to user
sudo mv kubectl oc openshift-install /usr/local/bin

# Check versions
openshift-install version

sudo rm /usr/local/bin/kubectl /usr/local/bin/oc
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux-amd64-rhel8.tar.gz
tar -xvf openshift-client-linux-amd64-rhel8.tar.gz
sudo mv kubectl oc /usr/local/bin
oc version
```


## 3. CA certificates
“OpenShift安装程序” 需要与 “vCenter API” 建立安全的 HTTPS 连接，没有受信任的证书，系统会认为连接不安全而拒绝通信

```bash
# 1. Download the certificate
wget --no-check-certificate https://ocpgym-vc.techzone.my.local/certs/download.zip
unzip download.zip

# 2. 复制证书文件到它该去的地方
sudo cp certs/lin/* /etc/pki/ca-trust/source/anchors

# 3. 生成系统证书捆绑文件，让系统识别新证书
sudo update-ca-trust extract
```

## 4. Create Config file
Create the `install-config.yaml` file

!!! warning
    Some installation assets, such as bootstrap X.509 certificates, have short expiration intervals, therefore you must NOT reuse an installation directory!


```bash
openshift-install create install-config --dir=ocp4

> SSH: /home/admin/.ssh/ocpgym.pub
> Target Platform: vsphere
> Domain/URL of the vCenter: `vsphere_hostname` in `/home/admin/vmware-ipi.yaml`
> The username to login to the vCenter: `vsphere_username` in `/home/admin/vmware-ipi.yaml`
> The password to login to the vCenter: `vsphere_password` in `/home/admin/vmware-ipi.yaml`
> Default Datastore: `vsphere_datastore` in `/home/admin/vmware-ipi.yaml`
> Virtual IP Address for OpenShift API: `vsphere_api_vip` in `/home/admin/vmware-ipi.yaml`
> Virtual IP Address for Ingress: `vsphere_ingress_vip` in `/home/admin/vmware-ipi.yaml`
> Base domain of the cluster(All DNS records will be sub-domains of this base and will also include the cluster name.): `base_domain` in `/home/admin/vmware-ipi.yaml`
> Cluster name: ocpinstall: `cluster_name` in `/home/admin/vmware-ipi.yaml`
> Pull secret: The container registry pull secret for this cluster, as a single line of JSON (e.g. {"auths": {...}}). You can get this secret from https://console.redhat.com/openshift/install/pull-secret
```

As Result, you will get this:

```txt
ocp4/
└── install-config.yaml
```

## 5. Modify Config for Master & Worker Nodes 
config Master Node(controlPlane) & Worker Node(compute)

<table>
  <tr>
    <th>Before</th>
    <th>After</th>
  </tr>
  <tr>
    <td>
    ```yaml
    controlPlane:
    architecture: amd64
    hyperthreading: Enabled
    name: master
    replicas: 3
    platform: {}
    compute:
    - architecture: amd64
    hyperthreading: Enabled
    name: worker
    replicas: 3
    platform: {}
    ```
    </td>
    <td>
    ```yaml
    controlPlane:
        architecture: amd64
        hyperthreading: Enabled
        name: master
        replicas: 3
        platform:
            vsphere:
            cpus:  8
            coresPerSocket:  2
            memoryMB:  16384
            osDisk:
                diskSizeGB: 120
    compute:
      - architecture: amd64
        hyperthreading: Enabled
        name: 'worker'
        replicas: 3
        platform:
            vsphere:
            cpus:  16
            coresPerSocket:  2
            memoryMB:  32768
            osDisk:
                diskSizeGB: 300
    ```
    </td>
  </tr>
</table>

## 6. Modify other config
change `machineNetwork`:

```yaml
networking:
  machineNetwork:
  - cidr: 10.0.0.0/16
```

* OpenShift set it defaultly to `10.0.0.0/16`, therefore we need to update it to what we want


## 7. Run installation
```bash
openshift-install create cluster --dir ./ocp4 --log-level=info
```

```txt
WARN[0000] vsphere authentication fields are now deprecated; please use vcenters
WARN[0000] vsphere topology fields are now deprecated; please use failureDomains
WARN[0000] computeCluster as a non-path is now deprecated; please use the discovered form: /myCloud/host/ocp-gym
WARN[0000] datastore as a non-path is now deprecated; please use the discovered form: /myCloud/datastore/693039cd8465ea1751697c4f-storage
INFO Successfully populated MCS CA cert information: root-ca 2035-12-03T14:05:48Z 2025-12-05T14:05:48Z
INFO Successfully populated MCS TLS cert information: root-ca 2035-12-03T14:05:48Z 2025-12-05T14:05:48Z
INFO Consuming Install Config from target directory
INFO Adding clusters...
I1205 09:16:17.788634   31401 session.go:227] "Created and cached vSphere client session" server="ocpgym-vc.techzone.my.local" datacenter="" username="itz-693039cd8465ea1751697c4f@vsphere.local"
INFO Creating infrastructure resources...
INFO Importing OVA ocpinstall-pqwhg-rhcos-generated-failure-domain into failure domain generated-failure-domain.
INFO Started local control plane with envtest
INFO Stored kubeconfig for envtest in: /home/admin/ocp4/.clusterapi_output/envtest.kubeconfig
INFO Running process: Cluster API with args [-v=2 --diagnostics-address=0 --health-addr=127.0.0.1:40023 --webhook-port=40493 --webhook-cert-dir=/tmp/envtest-serving-certs-1181241836 --kubeconfig=/home/admin/ocp4/.clusterapi_output/envtest.kubeconfig]
INFO Running process: vsphere infrastructure provider with args [-v=2 --diagnostics-address=0 --health-addr=127.0.0.1:36093 --webhook-port=34619 --webhook-cert-dir=/tmp/envtest-serving-certs-3908076429 --leader-elect=false --kubeconfig=/home/admin/ocp4/.clusterapi_output/envtest.kubeconfig]
INFO Creating infra manifests...
INFO Created manifest *v1.Namespace, namespace= name=openshift-cluster-api-guests
INFO Created manifest *v1beta1.Cluster, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-0
INFO Created manifest *v1beta1.VSphereCluster, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-0
INFO Created manifest *v1.Secret, namespace=openshift-cluster-api-guests name=vsphere-creds-0
INFO Done creating infra manifests
INFO Creating kubeconfig entry for capi cluster ocpinstall-pqwhg-0
INFO Waiting up to 15m0s (until 9:32AM EST) for network infrastructure to become ready...
INFO Network infrastructure is ready
INFO Created manifest *v1beta1.VSphereMachine, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-bootstrap
INFO Created manifest *v1beta1.VSphereMachine, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-master-0
INFO Created manifest *v1beta1.VSphereMachine, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-master-1
INFO Created manifest *v1beta1.VSphereMachine, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-master-2
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-bootstrap
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-master-0
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-master-1
INFO Created manifest *v1beta1.Machine, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-master-2
INFO Created manifest *v1.Secret, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-bootstrap
INFO Created manifest *v1.Secret, namespace=openshift-cluster-api-guests name=ocpinstall-pqwhg-master
INFO Waiting up to 15m0s (until 9:32AM EST) for machines [ocpinstall-pqwhg-bootstrap ocpinstall-pqwhg-master-0 ocpinstall-pqwhg-master-1 ocpinstall-pqwhg-master-2] to provision...
INFO Control-plane machines are ready
INFO Cluster API resources have been created. Waiting for cluster to become ready...
INFO Waiting up to 20m0s (until 9:45AM EST) for the Kubernetes API at https://api.ocpinstall.gym.lan:6443...
INFO API v1.33.5 up
INFO Waiting up to 1h0m0s (until 10:25AM EST) for bootstrapping to complete...
INFO Waiting for the bootstrap etcd member to be removed...
INFO Bootstrap etcd member has been removed
INFO Destroying the bootstrap resources...
INFO Waiting up to 5m0s for bootstrap machine deletion openshift-cluster-api-guests/ocpinstall-pqwhg-bootstrap...
INFO Shutting down local Cluster API controllers...
INFO Stopped controller: Cluster API
INFO Stopped controller: vsphere infrastructure provider
INFO Shutting down local Cluster API control plane...
INFO Local Cluster API system has completed operations
INFO no post-destroy requirements for the vsphere provider
INFO Finished destroying bootstrap resources
INFO Waiting up to 40m0s (until 10:20AM EST) for the cluster at https://api.ocpinstall.gym.lan:6443 to initialize...
INFO Waiting up to 30m0s (until 10:24AM EST) to ensure each cluster operator has finished progressing...
INFO All cluster operators have completed progressing
INFO Checking to see if there is a route at openshift-console/console...
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run
INFO     export KUBECONFIG=/home/admin/ocp4/auth/kubeconfig
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.ocpinstall.gym.lan
INFO Login to the console with user: "kubeadmin", and password: "xx-xx-xx-xx"
INFO Time elapsed: 38m51s
```

!!! warning
    万一安装失败，安装程序会删除 install-config.yaml 文件，该设计是出于安全，幂等性考虑


## 8. Login
`ocp4/auth/kubeconfig` 是一个自动生成的带有登陆密码的文件

```bash
# Test login with kubeconfig
export KUBECONFIG=ocp4/auth/kubeconfig

# Test login with User&PW
oc login -u kubeadmin -p $(cat ocp4/auth/kubeadmin-password) https://api.ocpinstall.gym.lan:6443
```