[TOC]

# 1. 卷（Volume）
Docker 容器本质上是 **瞬态（transient）**  的。 这意味着它们只能持续很短的时间。它们在App被创建时被调用，容器被销毁时，数据与容器一起被销毁。

为了使容器的数据**持久化**，我们将**容器**所在的 **Pod** 连接到**卷**。 容器中的数据放在该**卷**中，即使容器被删除，数据会仍然存在。

下面的yaml文件为**Pod**创建并连接**卷**：

- **Container**：其中的App随机生成一个0-100之间的数，并存到文件夹 `/opt` 中
- **Volume**：该**卷**名为`data-volume`，实际上是当前Node上的一个文件夹`/data`。所有用卷`data-volume`存储的信息，最后都会被放在Node的`/data`文件夹下

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    
    volumeMounts:       # (1) 将容器文件夹`/opt`挂载到名为data-volume的卷上
    - mountPath: /opt     # Container内部文件系统中的文件夹名
      name: data-volume   # Volume的名字
  
  volumes:              # (2) 创建卷：这里申明了一个在Node上的文件夹，也可以使用其他存储方式，比如引用（PVC）
  - name: data-volume     # Volume的名字
    hostPath:
      path: /data
      type: Directory
```
卷的使用确保了**Pod**被删除后，该**Pod**所产生的数据还被保留在所在的**Node**上，如图：
<img src="../ckad-7/volume.png" width=1000 />

上面图中的的挂载方法只适用于单Node的集群。如果是多个**Node**，那么每个**Node**其实都是不同的服务器，它们都有的`/data`文件路径，分别存储了不同的内容。所以我们要另找解决方法。

<img src="../ckad-7/multi_node.png" width=1200 />

- **卷**与**Pod**相连 -> 每次新建一个需要存储空间的**Pod**，都需要手动配置卷
- **卷**存在于**Node服务器**上 -> 分布在不同**Node**上的App无法访问到同一个的**卷**
- 卷的这两个特性导致管理困难，而且也不利于扩大应用规模，因为无法支持多Node的应用，而**持久卷**能解决该问题

<!--
## 常见的 volume 的类型
|Volume|`hostPath` |云储存|
|:-|:-|:-|
|**级别**| Pod级别的volume| 比如`awsElasticBlockStore`，数据相对Node和App独立，分布在不同Node上的应用都统一访问云存储的空间|
|**多Node应用**|不支持|支持|
|**yaml<br />举例**|<img src="../ckad-7/eb458cc117a449d39469c0ae5ebf212c.png" />|<img src="../ckad-7/664c8abed12a4d8aafd66763d6e23d54.png" />|
|||<img src="../ckad-7/cc7ff739460d481588c53b3c6fd81fd7.png" width=1200 />|

# TODO：比较hostPath和emptyDir
|Volume|`hostPath`|`emptyDir` |云储存|
|:-|:-|:-|:-|
|**级别**| Pod级别的volume| `emptyDir`在Pod被分配到一个Node上时自动生成，初始化后没有任何内容。是零时卷，是Pod级别的volume，与该volume相关联的Pod的所有Container都能访问到volume中的内容。当 Pod 因任何原因从节点中移除时，emptyDir 中的数据将被永久删除。| 比如`awsElasticBlockStore`，数据相对Node和App独立，分布在不同Node上的应用都统一访问云存储的空间|
|**多Node应用**|不支持|不支持|支持|
|**yaml举例**|<img src="../ckad-7/eb458cc117a449d39469c0ae5ebf212c.png" />|<img src="../ckad-7/605e97f2c9f94fe496b014ae7c7e7da3.png" />|<img src="../ckad-7/664c8abed12a4d8aafd66763d6e23d54.png" />|
**更多卷（`volume`）的类型见[这里](https://kubernetes.io/docs/concepts/storage/volumes/#awselasticblockstore)。*
-->


# 2. 持久卷（PersistentVolume / PV）
我们想要一个更中心化的解决方法，如有某个**多Node的应用**需要修改存储空间，管理员（Administrator）可以统一对其进行管理。**持久卷池**可以帮我们解决这个问题。

持久卷是**Cluster**层级的，放满 **存储卷** 的池子（Pool），由集群的管理员进行管理和配置。然后由Pod根据自己的需求发送 **存储卷请求（PersistentVolumeClaim / PVC）**。

⚠️ 一个PV本身是不可分割的单位，而管理员管理的是一堆存储容量大小各异的PV
```yaml
# 持久卷（PersistentVolume）的定义
# my-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  accessModes:    # 可能的mode： ReadOnlyMany, ReadWriteOnce 和 ReadWriteMany
    - ReadWriteOnce
  capacity:       # 所需的存储空间的大小
    storage: 1Gi
  hostPath:       # 卷的类型：Node（主机）上的文件夹 - 不推荐
    path: /tmp/data
```
生成持久卷：

```bash
kubectl create -f my-pc.yaml
```
查看已有的持久卷：
```bash
kubectl get persistentvolume
```

## 持久卷的类型
当我们在`pod.yaml`中定义`volumes`属性时，有很多不同选择。比如：NFS、ClusterFS、Flocker、FibreChannel、CephFS、ScaleIO 或公共云解决方案，如AWS EBS， Azure， Google Persistent Disk，或者AWS Elastic Block Store。
**- AWS Elastic Block Store：**
```yaml
volumes:
- name: data-volume
  awsElasticBlockStore:
  	volumeID: <volume-id>
  	fsType: ext4
```
还有其他更多**持久卷**的类型：

- `awsElasticBlockStore`：AWS Elastic Block Store (EBS)
- `azureDisk`：Azure Disk
- `azureFile`：Azure File
- `cephfs`：CephFS volume
- `csi`：Container Storage Interface (CSI)
- `fc`：Fibre Channel (FC) storage
- `gcePersistentDisk`：GCE Persistent Disk
- `glusterfs`：Glusterfs volume
- `hostPath`：HostPath volume (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using local volume instead)
- `iscsi`：iSCSI (SCSI over IP) storage
- `local`：local storage devices mounted on nodes.
- `nfs`：Network File System (NFS) storage
- `portworxVolume`：Portworx volume
- `rbd`：Rados Block Device (RBD) volume
- `vsphereVolume`：vSphere VMDK volume


# 3. 持久卷申领（PersistentVolumeClaim，PVC）
在定义了Cluster上的**PersistentVolume**之后。**Pod**需要发送PVC请求，以拿到试用PV的许可

!!! note "概念整理"
    - **PV**：由 **管理者（Administrator）** 配置的存储空间
    - **PVC**：由Pod的 **用户（User of Pod）** 发出的“想要使用一部分存储空间“的请求

```yaml
# my-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:        # 所需资源
    requests:
      storage: 500Mi
```
生成PVC：
```bash
kubectl create -f my-pvc.yaml
```
查看已有PVC：
```bash
kubectl get persistentvolumeclaim
```
PVC在Pod中的使用：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:  # 使用PVC
        claimName: myclaim
```
⚠️ 在Pod中添加PVC之后，ReplicaSets 和 Deployments中也会自动添加相对应的PVC定义！



## PVC如何选择PV
### 1. 根据属性选择
Kubernetes 尝试根据**PVC的要求**找到具有足够容量的PV。**PVC**可以定义存储容量（sufficient capacity），访问模式（access modes）、卷模式（volume modes）、存储类（storage class）等属性。如果有多个PV符合PVC的要求，则随机选择一个PV。

### 2. 根据selectors和labels选择
当然，如果想要绑定到特定的PV，也可以利用`labels`和`selectors`来定位到正确的PV。比如：
1. Pod中添加`selectors`属性
```yaml
# pod.yaml
selector:
	matchLabels:
		name: my-pv
```
2. 在PV中添加`labels`属性

```yaml
# pv.yaml
labels:
	name: my-pv
```

### 3. 特例
最后，如果所有所有属性都匹配，且没有更好的选择，则较小的PVC最终可能会绑定比它要求更大的PV。<br/>
**--> 这可能导致PV的浪费！**

假设我们有一个PVC和一个PV。PVC要求的存储空间是`500Mi`，而现在可用的PV只有一个，那么PVC只能绑定到剩下的PV。用`k get pvc`可用看到：
```bash
NAME     STATUS    VOLUME   	CAPACITY   ACCESS MODES   STORAGECLASS    AGE
my-claim Bound     my-volume    1Gi        RWO            aws-xxx		  40m
```

### 4. 没有可用的PV时，PVC的行为
如果当前的Cluster中，没有可用的PV，那么发出的PVC会一直处于`Pending`的状态。这时用`k get pvc`会看到：
```bash
NAME     STATUS    VOLUME   CAPACITY    ACCESS MODES   STORAGECLASS           AGE
my-claim Pending                                       vpc-block-1iops-tier   35h
```
当Administrator往Cluster中添加新的PV时，处于`Pending`状态的PVC会自动与新的PV完成绑定！

# 4. 删除PVC
删除PVC：

```bash
k delete pvc/my-claim
```
当PVC被删除时，**与之绑定的PV** 可能的行为有以下几种：

1. `Retain`: 被绑定的PV会依旧存在，没有被删除，除非Administrator手动删除。但该PV也无法被其他PVC使用。此时PV的Status会是`“Released”`，而不是`“Available”`。
2. `Delete`: 被绑定的PV也会自动被删除，由此，Cluster的整体**存储池**总体容量会变小
3. `Recycle`: 被绑定的PV被回收利用，PV上的数据会被删除，且可供其他PVC使用，Cluster的整体**存储池**总体容量会不变

你可以在YAML中定义：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  awsElasticBlockStore:
    volumeID: xxx
    fsType: ext4
  persistentVolumeReclaimPolicy: Recycle  # 对PV的行为进行规定："Retain", "Recycle", 或 "Delete"
```
⚠️ 对卷的绑定可以在Pod的定义文件中进行，也可以在Deployment或者ReplicaSet的定义文件中进行，效果是一样的

# 5. 存储类 / Storage Classes

假设我现在想使用GCE的存储空间，那么需要4个步骤：

1. 在GCE中新建存储空间
2. 创建一个使用GCE空间的PV
3. 创建一个PVC，对PV进行时使用
4. 在Pod中使用引用PVC

这个过程被称作 **Static Provisioning**。我们可以通过创建**存储类 / Storage Classes**来自动化步骤（1）和（2），实现**“Dynamic Provisioning”** 。Static和Dynamic Provisioning的区别如图：

<img src="../ckad-7/provisioning.png" width=600>


**1.创建一个存储类 / StorageClass**
```yaml
# my-sc.yaml
apiVerson: storage.k8s.io/v1
kind: StorageClass
metadata:
	name: google-storage

provisioner: kubernetes.io/gce-pd   # 配置器：不同供应商提供不同的配置器
VolumeBindingMode: WaitForFirstConsumer
parameters:             # 配置额外的参数
	type: pd-standard
	replication-type: none
```

!!! note "provisioner"
    注意，kubernetes提供了多个provisioner，为每个云供应商都提供了provisioner，`kubernetes.io/gce-pd`是给 GoogleCloudEngine 的provisioner

    是如果provisioner是`kubernetes.io/no-provisioner`，那么意味着该StorageClass不支持 dynamic provisioning

!!! note "绑定模式/VolumeBindingMode"
    可能的值：<br/>
    
    - `WaitForFirstConsumer`: 将延迟 PV 的绑定和配置，直到创建使用了对应 PVC 的 Pod <br/>
    - `Immediate`

**2.创建一个PVC, 使用StorageClass**
```yaml
# my-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi
```
**工作原理：**
PVC通过`storageClassName`连接到StorageClass，StorageClass中的 **配置器/provisioner** 来配置新的GCE磁盘，并且自动生成一个相对应的PV

**3.在Pod中使用该PVC**

# 6. Stateful Set
**Stateful Set** 类似于 **Deployment**，因为它们基于模板创建 Pod。 他们可以增加和减少Pod的数量。 也可以执行滚动更新和回滚，但存在差异：
**Stateful Set** 中 **Pod** 是按顺序创建的。 部署第一个 **Pod** 后，它必须处于运行和就绪状态，才能继续部署下一个 **Pod** 。 **Stateful Set**给每一个创建的**Pod**都按照生成顺序编号，并根据编号生成独一无二的名字，比如：`sql-1`，`sql-2`

```yaml
# Stateful Set的定义文件和Deployment大同小异，主要把 kind的类型改了
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  
  serviceName: mysql  # 声明要使用的 Headless服务
```
生成StatefulSet：

```bash
kubectl create -f my-ss.yaml
```

⚠️ StatefulSet被删除时从后往前删

# Headless Service
不做loading balance，只提供DNS的服务。与StatefulSet一起使用时，为每一个Pod都提供一个DNS，格式为：`<pod-name>.<headless-service-name>.<namespace>.<cluster-domain>`比如：

- `mysql-0.mysql-headless.default.svc.cluster.local`
- `mysql-1.mysql-headless.default.svc.cluster.local`
- `mysql-2.mysql-headless.default.svc.cluster.local`

```yaml
apiVersion: v1
kind: Service
metadata:
	name: mysql-headless
spec:
	ports:
		- port: 3306
	selector:
		app: mysql
	clusterIP: None  # Headless服务与其他服务区分开来的地方
```
现在我们有两条路：

1. 手动创建Pod
2. 通过StatefulSet创建Pod

## 1. 手动创建Pod
```yaml
apiVersion: apps/v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: mysql
    image: mysql

  subdomain: mysql-headless   # 1. subdomain = 服务名字
  hostname: mysql-pod         # 2. hostname 确保每个Pod都有各自的DNS
```

## 2. 通过StatefulSet创建Pod
如之前的例子所示，需要加一个`serviceName`即可

#  >>>  本章kubectl命令整理
**PersistentVolume相关**

`k get pv`

-- --

**PersistentVolumeClaim相关**

`k get pvc`

`k delete pvc/my-claim` 删除pvc

# 图片来源
<a href="https://www.flaticon.com/free-icons/database" title="database icons">Database icons created by phatplus - Flaticon</a>
<a href="https://www.flaticon.com/free-icons/folder" title="folder icons">Folder icons created by Good Ware - Flaticon</a>
