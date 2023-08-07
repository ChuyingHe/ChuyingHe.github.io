[TOC]

# 1. 考试形式

- 远程，详细要求可以在官网上找到
- 考试时可用的资源：k8s 官方文档

# 2. 基本概念

- **容器（container）**：包含了 App，和运行 App 所需的所有配置与环境。比起虚拟机（VM）来更高效，更灵活
- **Pod**：是 k8s 中的最小单位，Pod 中的内容分享了**同样的网络（Network）和存储（Storage）**。Pod 封装了一个或多个 container（如果是多个 container，那么大概率这些 container 包含了不同的 App，比如一个前端 App，一个数据库 App）。豆荚也可翻译成 pod，所以可以联想到图片如下：<br />

<img src="../ckad-1/pod.jpeg" width=300 />

⚠️ Pod 常被用于扩大应用的规模，假设你的 App 访问人数在某个节假日激增，那我们就新建一个 pod，然后把同样的 App 部署到新建的 pod 上；同样的，Pod 也可以被用于降低 App 的规模

- **节点（node）**：安装了 k8s 的机器，Pod 就是在 node 上跑的。（节点出问题被自动关闭的时候，其上面的容器中的 App 自然也会被关闭，所以一般我们会**_多节点运行_**）
- **工作节点（worker node）**：真正存放容器，干活的节点
- **master 节点（master node）**：也被叫做 **控制面板/Control plane**，监视集群中的 worker nodes，并负责对 worker nodes 上的容器进行实际编排。一个 cluster 中可以有一个或者多个 master nodes
- **集群（cluster）**：由一群 nodes 组成

<img src="../ckad-1//844171d8f36a4d5c88e1c1883ada9aa6.png">

## Master node VS Worker node

|            | Master node<br/>/Control plane | Worker node           |                                                                                                                                                                                                                    |
| :--------- | :----------------------------- | :-------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Server     | 带`kube-apiserver`服务         | 带`kubelet agent`代理 | `kube-apiserver`与`kubelet agent`之间有相互的沟通，<br /> 比如`kubelet`提供 worker node 是否健康的信息，<br /> `kube-apiserver`给`kubelet`发送任务信息等等。<br /> 所有信息都存储在 master node 上的 etcd 数据库中 |
| ETCD       | ✓                              | ✗                     |                                                                                                                                                                                                                    |
| Controller | ✓                              | ✗                     |                                                                                                                                                                                                                    |
| Scheduler  | ✓                              | ✗                     |                                                                                                                                                                                                                    |

# 3. k8s 的六大组件

<img src="../ckad-1/components-of-kubernetes.svg">
当你安装k8s时，你实际上安装了以下组件：
## (1) API Server
相当于k8s的前端，所有的users，devices，CLIs都通过API Server与k8s集群进行沟通。
## (2) ETCD Service
（发音为et-see-dee)，是一个存储***键值对（key-value）*** 的数据存储，用于实现node的锁，以防止冲突的产生。
## (3) Kubelet Service
是在cluster中每个node上运行的代理（agent）。负责确保按预期在节点上运行。
## (4) Container Runtime
是用于运行容器的底层软件。通常这个软件是 Docker，但也有其他容器技术。
## (5) Controllers
控制器（Controllers）负责编排（orchestration）。负责在node、container或endpoint出现故障时进行通知和响应。（比如，在特定情况下，Controller会决定启动新容器）。在k8s中有多个Controllers，每个负责不同的工作。
## (6) Scheduler
负责nodes之间的工作分配和调度。如果有新建的container，Scheduler会将container分配给node

# 4. k8s 中的 YAML

`YAML`基本介绍见[3 分钟看懂 YAML](http://t.csdn.cn/ZRc2c)。k8s 中，我们可以选择用`k`命令行工具来部署应用，也可以用`YAML`文件定义”如何部署应用“，然后再一次性将`YAML`文件执行到 k8s 集群上。相比`k`，使用`YAML`的优点有：

- Devops as code：可以用 git 对`YAML`进行版本控制，像写软件代码一样写 devops 的部署，且多个程序猿可同时进行协作，不会出现信息不同步的问题
- Single source of truth：所有和 devops 相关的信息都可以查看对应的`YAML`文件
- 容易 debug
- 容易进行项目重建：假设你要把现有的部署挪到另一个云供应商平台上去，只需要在部署一遍`YAML`文件即可

在 k8s 中的`YAML`文件都默认应该有**四个属性**：

- `apiVersion`：版本，可以是 v1 或者 apps/v1
- `kind`：资源种类，可以是 Pod，Service，ReplicaSet 或 Deployment 等
- `metadata` ：该资源的元数据，包括 name，labels 等字典类型数据，不允许其他键的存在
- `spec`：配置信息，每个资源应有的 spec 都不一样

以下是一个 Pod 的`YAML`文件的例子：

```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
	labels:						# s在`lables`中，你可以定义任意到键值对，以帮助你之后快速地找到相关资源
		app: myapp
		type: frontend
		author: chuuuing
spec:
	containers:
		- name: nginx-container
		  image: nginx
```

接下来我们将该`YAML`文件部署到集群上：

```bash
kubectl create -f pod-definition.yml
```

## apiVersion

| `kind`（资源种类） | `apiVersion`（版本） |
| :----------------- | :------------------- |
| Pod                | v1                   |
| Service            | v1                   |
| ReplicaSet         | apps/v1              |
| Deployment         | apps/v1              |

# 5. Replication Controller

`Replication Controller`是 Controller 组件的一种。

- 监控并确保自己负责的`Pod`数量符合要求（比如：`Pod`的数量多了那就杀掉几个，少了就自动生成几个）
- 扮演一个 load balancer 的角色，跨 Node 平衡访问

!!! note
`Replication Controller` vs `Replica Set` <br />
两者都有上述提到的功能。**Replica Set**是更新的概念，多一个`.spec.selector`的属性，背后的原因是： <br /> - 除了由自己拷贝的`Pod`副本，**Replica Set**也可以用于管理其他`Pod`； <br /> - 而**Replication Controller**默认只能管理自己生成的`Pod`副本。 <br />

```yaml
apiVersion: app/v1
kind: ReplicaSet
metadata:
	name: myapp-replicaset
	labels:
		app: myapp
		type: frontend
spec:
	template:
		metadata:
	  		name: myapp-pod
		  	labels:
		    	app: myapp
		    	type: frontend
		spec:
			containers:
				- name: nginx-container
	  		  	  image: nginx
	replicas: 3

	selector:			# ReplicaSet独有的，ReplicationController没有的！
		matchLabels:
			type: frontend
```

<table>
    <tr>
        <th>不带`selector`</th>
        <th>带`selector`</th>
    </tr>
    <tr>
        <td>监视器代表 ReplicationController<br/>只默认管理由自己创建的Pod</td>
        <td>监视器代表 ReplicaSet<br/>可以管理所有带`matchLabels`的Pod</td>
    </tr>
    <tr>
        <td><img src="../ckad-1/f4e1a8f0e24445cd97eb49d550f7a9f8.png" width=370 /></td>
        <td><img src="../ckad-1/39cb34bbd5ee4a11b19724ce6ffc2db0.png" width=390 /></td>
    </tr>
</table>

<!--
1）把当前`replicaset`的yaml保存起来`k get replicaset myapp-replicaset -o yaml`
2）修改该`yml`文件中的relicas的数量
3）再用强制取代旧的`replicaset`：`k replace -f xxx.yml` （`-f`代表`force`）
 **方法二**：假设当前`replicaset`也是用yaml文件建立起来的，`k scale --replicas=6 -f xxx.yml` 命令会同时修改YAML文件，并更新部署-->

**如何 scale up/down 集群中`Pod`的数量？**

- **方法一**：`k edit replicaset`直接修改 yaml 格式
- **方法二**： `k scale --replicas=6 replicaset myapp-replicaset` 只更新部署，YAML 文件不会被自动修改
- **其他**： 根据用户流量自动 scale，我们之后会讲到

**如果`ReplicaSet`中的 container template 有错，比如 image 的名字错了，如何修改？**

- **方法一**：删除并重建`ReplicaSet`（删除`ReplicaSet`会自动删除它所监控的 Pod）
- **方法二**：先更新`ReplicaSet`，删除旧的 Pod `k edit replicaset xxx` ；然后 `k delete pod -l name=busybox-pod`

!!! note
因为`ReplicaSet`只检查数量，不检查 Pod 的内容，所以要把旧的 Pod 杀掉

!!! warning
**ReplicaSet** 的`apiVersion`的值是`apps/v1`，不是`v1`，不然你会看到一下错误：

    	![请添加图片描述](../ckad-1/123a2143752c4f6095b937ffc734a84b.png)

# 6. Deployment

迄今为止，我们知道`Container`跑在`Pod`上，而`Pod`由`ReplicaSet`监控，`Deployment`被看作在`ReplicaSet`外的另一层外套。`Deployment`提供更新，撤消更新回滚到旧版本（rolling），暂停和恢复更改等功能。

```yml
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: myapp-deployment
	labels:
		app: myapp
		type: frontend
spec:
	template:
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: frontend
		spec:
			containers:
			- name: nginx-container
			  image: nginx
	replicas: 3
	selector:
		matchLabels:
			type: frontend
```

!!! note "Deployment vs ReplicaSet"
注意到了吗，除了`kind`，其他内容和`ReplicaSet`中没有区别！
这时候我们用`k create -f deployment.yml`创建 `Deployment`，你可以看到集群中会自动新建以下资源：

    	- Deployment
    	- ReplicaSet
    	- Pod

🪆 因为他们之间的关系是一层套一层：`Pod` < `ReplicaSet` < `Deployment`，像俄罗斯套娃一样：
![请添加图片描述](../ckad-1/0cb1cb71c50849c2aee56e0593098236.jpeg)

**用 kubectl 新建 deployment：**
用镜像新建一个 Deployment:

```bash
kubectl create deployment [DeploymentName] --image=[ImageName]
```

scale up:

```bash
kubectl scale deployment --replicas=3 [DeploymentName]
```

# 7. Namespace

当集群建立起来的时候，k8s 会自动生成三个默认的 Namespace：

1. `default`：默认的 Namespace，如果你不另外创建专门给 App 的 Namespace 的话，一进集群就默认使用`default`
2. `kube-system`：k8s 自动生成一些供内部使用的 Pods 和 Service，比如 DNS 服务。这个 Namespace 的存在保证了用户不会破坏这些内部资源
3. `kube-public`：这里应该放一些 **所有 Namespace** 都有权限访问的资源

> 每个`Namespace`都有自己的参数设置和限制，能保证不占用过多的资源（比如说储存空间等）。`Namespace`常见的应用场景是：一个`dev`给开发，一个`prod`的`Namespace`在使用中。

生成 pod.yaml 时，我们也可以指定该 Pod 所在的 Namespace：

```yaml
...
metadata:
	name: myapp-pod
	# 指定 Namespace
	namespace: dev
	labels:
		app: myapp
		type: frontend
...
```

用 YAML 新建 Namespace：

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

或者，用`k`新建：`k create namespace dev`

**其他相关的`k`的命令**

- `k config current-context` 查看当前 context: `<用户名>:<cluster名>`
- `k get pods --namespace=xxx` 指定看 Namespace `xxx` 下的 Pod
- `k get pods --all-namespaces` 或者`k get pods -A` 查看所有 Namespace 下的 Pod
- `k config set-context --current --namespace=xxx` 把默认 namespace 设置成`xxx`

## Namespace 内部访问 vs 跨 Namespaces 之间的访问

![请添加图片描述](../ckad-1/8b7e0f79b6e742969e7ad33fa497ec7e.png)

Namespace 内部服务之间的服务访问 - 图中的(1）：

```bash
mysql.connect("db-service")
```

跨 Namespace 的服务访问 - 图中的(2）：**default** 访问 **dev** 中的数据库服务:

```bash
mysql.connect("db-service.dev.svc.cluster.local")
```

!!! note "跨 Namespace 的服务访问"
命名格式是`[ServiceName].[Namespace].svc.cluster.local`
-- --
**为什么可以这样访问到 Service 呢？**

    当一个Service被创建的时候，k8s会自动添加对应的DNS: <br/> `cluster.local`是k8s集群的默认域名（cluster domain），`svc`是子域名，`[Namespace]`是该Service所在的Namespace，`[ServiceName]`是Service本身的名字

## ResourceQuota

`ResourceQuota`用于给 **整个`Namespace`的总资源** 设限。比如`Pod`的数量，`CPU`数量，内存大小等等。
用 YAML 新建 ResourceQuota：

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

举例：`limits.cpu: "10"` 表示当前 Namespace 中所有 non-terminal 状态的 Pod 的`.limits.cpu`资源的总和没有超过 10

!!! note "terminal / 终止状态 的 Pod"
Pod which has .status == Failed or .status=succeeded

# >>> 本章 kubectl 命令整理

**新建容器：**

`k run my-new-pod --image nginx`

用镜像 nginx 部署一个容器到 kubernetes 集群上。因为 k8s 中最小单位是 Pod，所以同一时间，一个 Pod 也被生成

---

**彩排，输出 yaml 文件：**

`k run my-new-pod --image nginx --dry-run=client -o yaml > pod.yaml`

该命令不会马上执行“新建容器”的操作，`--dry-run=client`的意思是：我彩排一下，不真跑，只是看看是否可以创建资源，以及所用的命令是否正确。然后再用`-o yaml > pod.yaml`将彩排得到的内容以`yaml`文件的形式输出

---

**信息**

`k get all` 查看当前集群中所有

`k cluster-info` 查看当前集群信息

`k get nodes` 列举当前集群中所有的 nodes

---

**Pod 相关：**

`k run yyy --image=xxx` 用镜像`xxx`创建名为`yyy`的容器，放进 Pod 中，因为 k8s 中最小单位是 Pod！！没有办法把 Container 单独拿出来

`k get pods` 列举当前集群中所有的 pods

`k describe pod xxx` 打印名为`xxx`的 pod 的具体信息

`k edit pod xxx` 对已存在的 pod 进行修改

`k delete pod -l name=busybox-pod` 删除所有标签为`name=busybox-pod`的 Pod

---

**ReplicaSet 相关：**

`k get replicaset` 查看当前集群的 replicaset

`kl delete replicaset xxx` 删除名为`xxx`的 replicaset

`k edit replicaset xxx` 修改名为`xxx`的 replicaset

**修改 ReplicaSet 中 Pod 的数量：**

- 先修改`xxx.yml`文件中的 relicas 的数量，再用`k replace -f xxx.yml` 将更新部署到集群上
- `k scale replicaset --replicas=6  [ReplicaSetName]` 或 `k scale deployment --replicas=3 [DeploymentName]` 只更新部署，毋需 YAML 文件

---

**Deployment 相关**

`k create deployment [DeploymentName] --image=[ImageName] --replicas=4`

---

`k create -f xxx.yml` 将定义好的 yaml 文件部署到当前集群上

---

**kubectl 输出格式**

`k [command] [TYPE] [NAME] -o <output_format>`

- `-o json` 输出一个 JSON 格式的 API 对象。
- `-o name` 仅打印资源名称，不打印其他内容。
- `-o wide` 带附加信息的纯文本格式 ❗️
- `-o yaml` 输出一个 YAML 格式的 API 对象。

---

**namespace 相关**

`k config current-context` 查看当前 namespace

`k get namespaces ` 或`k get ns `

`k create namespace xxx`用`k`新建 Namespace

`k get pods --namespace=xxx` 指定看`xxx`Namespace 下的 Pod

`k get pods --all-namespaces` 查看所有 Namespace 下的 Pod

`k config set-context --current --namespace=xxx` 把默认 namespace 设置成`xxx`

---

**Service 相关**

`k expose pod redis --port=6379 --name redis-service` 为 Pod `redis` 新建一个名为`redis-service`的 ClusterIP 服务（`--type=ClusterIP`是默认值），该服务本身的端口是`6379`，该服务会用 Pod `redis` 的标签进行资源筛选

`k create service clusterip redis-service --tcp=6379:6379` 新建一个名为`redis-service`的 ClusterIP 服务，该服务将使用`6379`端口。该服务默认用`app=redis-service`标签进行资源筛选

---

直接暴露某个 Pod：

`k run XXX --image=XXX --port=80` 只会定义 port，不会真正的 expose Pod，也不会生成对应的 Service

`k run XXX --image=XXX --port=80 --expose=true` 会同时生成 Pod 和对应的 Service (默认为 ClusterIP)

# >>> 课后小笔记

## DNS

人类用姓名来标记和识别某个人，电脑则用 IP 地址。DNS（Domain Name System）是将网址（`www.baidu.com`）转换成 IP 地址（`103.235.46.40`）的桥梁，相当于一个电话簿，存的姓名是网址，而电话号码则是 IP 地址。

用 `dig` 命令你可以看到百度网址的 IP 地址。在浏览器中输入`www.baidu.com`或者`103.235.46.40`得到的结果是一样的：你都会看到百度搜索引擎首页。

```bash
dig www.baidu.com
```

!!! note "DNS"
<img src="../ckad-1/7fd8c9755eaf405cbf7ad062b99ed47c.png" width="600" />

    **示意图中发生的对话如下：**

    - **Laptop**：我要访问`www.baidu.com`，请问这个网址的IP地址是什么？
    - **Resolver**：你好，我是你的本地电话簿，我去看看我有没有存过这个网址。。。我没有存过，不知道它的IP是什么，我得去问问其他人
    - **Resolver**：你好**Root Server**，你知道网址`www.baidu.com`的IP地址是什么吗？
    - **Root Server**：我不知道，但我猜TLD应该知道，你去问他吧
    - **Resolver**：你好**TLD**，你知道网址`www.baidu.com`的IP地址是什么吗？
    - **TLD**：我不知道，但**SLD**应该知道
    - **Resolver**：你好**SLD**，你知道网址`www.baidu.com`的IP地址是什么吗？
    - **SLD**：我知道，是`103.235.46.40`
    - **Resolver**：感谢，我存下来，以防下次还需要用
    - **Resolver**：**Laptop**，你要的IP地址是`103.235.46.40`
    - **Laptop**：🙏


    (TLD = Top Level Domain) <br/>
    (SLD = Second Level Domain)

<!-- TODO: relationship between TLD, SLD and Root server -->
