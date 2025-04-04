# 1. 服务（`Service`）
!!! warning "Service vs Route"
	The `Service` will expose your application internally within the cluster, and the **Route** will create a publicly accessible URL for external access.


## 为什么需要Service？
Kubernetes中的每个`Pod`都有它自己的IP地址，但`Pod`经常会因为各种原因而挂掉，如果我们用`Pod`的IP地址来访问该`Pod`，会常出现IP地址无效的情况，我们用 **服务（`Service`）**  解决这个问题。**服务（`Service`）**提供了稳定的IP地址，我们永远可以找到**服务（`Service`）**的IP地址，然后通过该**服务（`Service`）**访问到想要找的应用Pod。

<div style="background-color: #4051b5; color: white; font-weight: 600;">服务不存在时：</div>
为了更好的理解，我们看一下如果 **服务不存在** ，用户可以怎么访问Pod中的应用：<br /> **方法一：** 如果用户本身在Cluster内部，可以直接通过`curl http:10.244.0.2`访问应用<br />  **方法二：** 如果在Cluster外部，可以通过 **ssh** 来访问`curl http:10.244.0.2`

<img src="../ckad-6/76a2876bc5f44e90b058f67b1017e627.png" width=600 />


<div style="background-color: #4051b5; color: white; font-weight: 600;">有了服务之后：</div>
Cluster外部对应用的访问会更方便。服务（`Service`）相当于一个虚拟服务器，有自己的IP地址（这里是`10.106.127.123`）。<br /> 这里我们不再需要 **ssh** ，而是直接访问Node所在的ip地址`192.168.1.2`<small>（⚠️用户本身的电脑IP和Node的IP是在同一个网络里面的，即IP地址中的前三部分时相同的，所以我们才能直接访问到Node得IP）</small>，<br /> 入口则使用Service提供的端口（`30080`），比如使用`curl 192.168.1.2:30080`
<img src="../ckad-6/3c7937f3bc754e5689a79e9f06fc360d.png"  width=600 />


查看服务：
```bash
k get services
# 或
k get svc
```
结果如下：
```bash
NAME			TYPE		CLUSTER-IP		EXTERNAL-IP 	PORT(S)			AGE
kubernetes   	ClusterIP   10.43.0.1    	<None>        	443/TCP   		7m30s
myapp-service	NodePort	10.106.127.123	<None>			80:30008/TCP	30s
```

!!! info
	- `kubernetes` 服务是k8s在启动时创建的默认服务。`myapp-service`服务是我们创建的，现在作为用户可以通过`curl http://192.168.1.2:30008`直接访问到前端App了！
	- PORT(S)中代表的是：`<ServicePort>:<NodePort>/<Protocol>`
	- `targetPort` 不会显示在 `oc get svc` 结果中
	- `ClusterIP`（集群内的虚拟 IP），集群内的 Pod 通过此 IP 访问 Service
	



## 语法：三个ports
```yaml
# Service with NodePort
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30007
```
YAML文件中共有三个端口：

1. `port` - Service 自己的 Port <br/> 
	- 服务本身的端口
	- 服务自己的port就简单地叫`port`
	- ✅ 必写，否则会报错
2. `targetPort` <br/> 
	- 应用程序的实际端口（Pod的端口） 
	- 是服务的目标，所以叫`target`
	- 🚫 可选，如不写明，默认与`port`值相同
3. `nodePort` <br/>
	- Node的端口，是外部访问服务的端口（集群外）
	- 🚫 可选，如不写明，k8s会提供任意一个在 **[30000, 32767]** 范围内可用的端口

<img src="../ckad-6/nodeport.png" width=500 />

如图，外部的访问者只需要知道Node的IP地址（`192.168.1.2`），和Node开放的Port（`30007`）就行了
	
!!! note "`service.spec.ports`"
	`.spec.ports`容纳的是一个数组，意味着一个服务可开放 **多个端口**，用于与不同的App进行沟通
	
	- ⚠️ 注意：如果有多个端口开放，那么`Service`的每个端口都必须有一个名称
	
	
	使用场景: 

	1. 多协议服务（如 HTTP + gRPC）- L7 应用层协议<br/>
		这里的 HTTP 和 gRPC 都是基于 TCP 的应用层协议。它们 使用不同的端口，但都是 TCP，例如：
		```yaml
		spec:
			ports:
			- name: http
				port: 80
				targetPort: 8080
			- name: grpc
				port: 50051
				targetPort: 50051
		```
	2. TCP + UDP - 例如DNS服务 - L4 传输层协议<br/>
		这里涉及 不同的传输层协议，即 TCP 和 UDP，所以必须分开声明
		```yaml
		spec:
			ports:
			  - name: dns-tcp
				port: 53
				targetPort: 53
				protocol: TCP
			  - name: dns-udp
				port: 53
				targetPort: 53
				protocol: UDP
		```
	3. 同时提供多个版本的 API
		```yaml
		spec:
			ports:
				- name: api-v1
				port: 8080
				targetPort: 8080
				- name: api-v2
				port: 9090
				targetPort: 9090
		```
	4. Service 代理多个不同的 Backend 端口. 一个 Pod 运行多个进程，每个进程监听不同端口（如 Web + Metrics）。
		```yaml
		spec:
		ports:
		  - name: web
			port: 80
			targetPort: 8080
		  - name: metrics
			port: 9090
			targetPort: 9090
		```




!!! info
	- `.spec.selector` 指定`targetPort`属于哪一个Pod，上面例子中绑定了一个`labels`中有`app=myapp`的Pod。
		- 很多情况下，一个前端App会有多个replicas，但因为他们都有相同的`labels`，他们会自动被service找到！
	- `targetPort`是Pod而不是Container！因为Pod中的所有Container是共享网络的，所以用同一个Pod端口



!!! note "服务的端点（Endpoint）"
	`Service`中有一个`Endpoint`，这个`Endpoint`是 该`Service` 指向的 真正的 Pod IP 地址和端口

	查看`Service`细节:
	
	```bash
	k describe svc <ServiceName>
	```

	<img src="../ckad-6/endpoint.png" width=400/>

## IP地址类型


有两种类型：**外部IP地址（external/public）** 和 **内部IP地址（internal/private）**。一般来说，每个设备都有自己的 **外部IP** 和 **内部IP**，且一般两者不同。

- **外部IP**：由你的网络供应商提供，你访问一个网页时（比如百度），百度可以看到你的外部IP地址，知道你是从哪儿来的。<br/>
	<pre><code>
	# ifconfig=InterFace config
	curl <span style="background-color: #ccd1f0">if</span>config.me
	</code></pre>


- **内部IP**：是自动生成的。人如其名，只做内部使用。比如，用于在家庭网络中识别你的身份信息。<br/>
	<pre><code>
	# ipconfig=Internet Protocol configuration
	<span style="background-color: #ccd1f0">ip</span>config getifaddr en0
	</code></pre>
	
!!! note
	当你使用VPN（Virtual Private Network）的时候，你的 **网络地址** 是被隐藏的
	
<img src="../ckad-6/28a6f7c5d63b4ccb836b22f3990464fb.png" width=500 title="source: https://www.avast.com/c-what-is-an-ip-address" />

## 服务类型
<table style='font-size: 10px'>
    <tr>
        <th style="width:3%">服务类型</th>
        <th style="width:40%">描述</th>
        <th style="width:20%">比喻</th>
        <th>YAML</th>
    </tr>
    <tr>
        <td><code>ClusterIP</code> <br/>❗默认类型</td>
        <td>默认类型，仅能在集群内部访问。比如：Frontend访问Backend<br/><br/><b>适用于</b>：集群内部服务间通信。</td>
        <td>公司内部的内线电话，外部无法拨打。</td>
        <td>
<pre><code>
spec:
	type: ClusterIP
</code></pre>
		</td>
    </tr>
    <tr>
        <td><code>NodePort</code></td>
        <td>暴露服务到每个节点的固定端口上，通过 <code>[NodeIP]:[NodePort]</code> 访问。<br/><br/><b>适用于</b>：测试环境，或快速暴露服务 - 用于外部访问。</td>
        <td>企业大门的访客入口，每个门有固定门牌号。</td>
		<td>
<pre><code>
spec:
	type: NodePort
	ports:
		- nodePort: 30007
</code></pre>
		</td>
    </tr>
    <tr>
        <td><code>Headless</code></td>
        <td>一种特殊的 <code>ClusterIP</code> 配置，无负载均衡，直接返回 Pod 的 IP，ClusterIP的集群 IP 为 None。<br/><br/><b>适用于</b>：直接访问 Pod 的场景，例如 StatefulSet 应用（如数据库）。</td>
        <td>给客户直接提供员工手机号码，而非总机转接。</td>
		<td>
<pre><code>
spec:
	clusterIP: None
</code></pre>
		</td>
    </tr>
    <tr>
        <td><code>LoadBalancer</code></td>
        <td>通过云平台创建一个 **外部负载均衡器**， 并分配公共 IP。<br/><br/><b>适用于</b>：Production环境中，那些需要直接对外暴露的服务。</td>
        <td>-</td>
		<td>
<pre><code>
spec:
	type: LoadBalancer
</code></pre>
		</td>
    </tr>
    <tr>
        <td><code>ExternalName</code></td>
        <td>将服务映射到外部的 DNS 名称，无需代理流量。<br/><br/><b>适用于</b>：使用外部资源（如外部数据库或 API）。</td>
        <td>-</td>
		<td>
<pre><code>
spec:
	type: ExternalName
	externalName: example.com
</code></pre>
		</td>
    </tr>
</table>






### （1）ClusterIP类 服务
!!! danger
	The name "ClusterIP" refers to the fact that this service provides a virtual IP address (ClusterIP) that is accessible only within the cluster.

默认类型，仅能在集群内部访问。比如：Frontend访问Backend

```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

!!! info
	问：假设现在有前端后端组成的程序，而前端和后端各有自己的`Replicas`（复制品，为了处理较大的用户访问而存在）。访问前端1的时候，前端1的API是发送给后端的哪一个`replica`的呢？

	答：Kubernetes的**ClusterIP服务**可以帮助我们将 `Pod` 分组在一起（所有的前端`replicas`为一组，后端`replicas`为另一组），并提供单个接口（即`Service`）来访问组中的 `Pod`。`Pod`会因为各种bug生生死死，每次重生之后拿到的IP地址还不一样，所以我们没有办法直接用`Pod`，因为我们不知道该去哪个IP地址找，而`Service`提供了稳定的IP地址。如图：
	
	<img src="../ckad-6/6c0afcac50c4444a96129c530c4da11f.png"  />


### （2）NodePort类
!!! danger
	The name "NodePort" refers to the fact that the service opens a specific port on each cluster node, making it accessible from outside the cluster via the node's IP and the exposed port.

暴露服务到每个节点的固定端口上，External clients can access it via `<NodeIP>:<NodePort>`.

NodePort 服务例子：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30007
```


!!! info "多Node"
	当Pod的replicas分布在不同的Node上时，Service会自动扩展到多个Node上，我们不需要做任何多余的操作。
	```bash
	curl http://192.168.1.2:30008
	curl http://192.168.1.3:30008
	```
	<img src="../ckad-6/a2d56d9a1dfa4a8a8e93316212828006.png" width=700 />


### （3）Headless类 
!!! danger
	The term "headless" implies that the service does not have a "head" or a centralized virtual IP address (ClusterIP). -> Instead, it directly exposes the backend pod IPs to clients.

当我们想要访问指定的Pod时，例如 StatefulSet 应用（如数据库）。只要将`.spec.clusterIP`设置为`None`即可：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
  namespace: default
spec:
  clusterIP: None  # Specifies headless
  selector:
    app: cassandra
  ports:
    - port: 9042  # Cassandra default port
      name: cql
```

!!! info
	大多数Stateful 的应用（比如数据库）来说，`Pod`的`Replicas`之间是有区别的。

	比如一个MySQL的应用可以分为`Master Pod`和`Worker Pod`：

	- 只有`Master Pod`可以向数据库写入数据
	- `Worker Pod`只负责同步和读取数据。

!!! info "那么如何才能访问到指定的Pod呢？"
	the headless service (defined with `clusterIP: None`) is responsible for creating DNS records for each individual pod in a StatefulSet. In our case we have a stateful app `Cassandra`:
	```yaml
	apiVersion: apps/v1
	kind: StatefulSet
	metadata:
	name: cassandra
	namespace: default
	spec:
	serviceName: cassandra  # Headless service name
	replicas: 3
	selector:
		matchLabels:
		app: cassandra
	template:
		metadata:
		labels:
			app: cassandra
		spec:
		containers:
			- name: cassandra
			image: cassandra:4.0
			ports:
				- containerPort: 9042  # Cassandra query language (CQL) port
			volumeMounts:
				- name: cassandra-data
				mountPath: /var/lib/cassandra
			env:
				- name: CASSANDRA_CLUSTER_NAME
				value: "CassandraCluster"
				- name: CASSANDRA_SEEDS
				value: "cassandra-0.cassandra.default.svc.cluster.local"
	volumeClaimTemplates:
		- metadata:
			name: cassandra-data
		spec:
			accessModes: ["ReadWriteOnce"]
			resources:
			requests:
				storage: 5Gi

	```
	
	the Service auto-generated these `DNS` for each pods:
	```bash
	cassandra-0.cassandra.default.svc.cluster.local
	cassandra-1.cassandra.default.svc.cluster.local
	cassandra-2.cassandra.default.svc.cluster.local
	```




### （4）LoadBalancer类 
!!! danger
	The name "LoadBalancer" signifies that this service type integrates with an external load balancer to distribute incoming traffic across the pods

`LoadBalancer`类Service = `NodePort`类Service + 发送一个 **添加负载均衡器（Load Balancer）** 的请求

```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
  namespace: default
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

## 创建服务（`Service`）
```bash
# 方法一：用yaml文件定义
k create -f my_service.yaml

# 方法二：生成service
# - port:TargetPort=8080:8080
# - 需要手动往YAML中修改`.spec.selector`
# - 自动生成了`.spec.ports.name`
k create service nodeport [ServiceName] \
	--tcp=8080:8080 \
	--dry-run=client \
	-o yaml > service.yaml
	
# 方法三：暴露deployment（以生成service）
# - 需要手动往YAML中添加`.metadata.labels`
k expose deployment/[DeploymentName] \
	--name=[ServiceName] \
	--port=8080 \
	--target-port=8080 \
	--type=NodePort \
	--dry-run=client -o yaml > service.yaml
```
<!-- TODO：截屏 -->

# 2. Ingress
<!--
	TODO

### Openshift的Route vs Kubernetes的Ingress
Openshift的`Route`诞生于k8s的`Ingress`之前，红帽作为k8s的主要贡献者之一，在很大程度上影响了`Ingress`的设计，可以说`Ingress`借鉴了`Route`的功能。
在k8s创建了`Ingress`之后，红帽为Openshift也设置了相关命令，现在在Openshift会监视`Ingress`对象，并创建一个或多个`Route`以满足`Ingress`指定的条件。
➡️ 如果在Openshift中创建`Ingress`对象，则无需再担心`Route`，因为在这种情况下，对应的`Route`会自动生成。
-->

## 背景
**🤔 当App在本地服务器上时，如何实现用域名 `http://www.my-shop.com`访问到`192.168.1.2:30008` 呢？**

- -> NodePort类的Service暴露 Node端口 30080
- -> 192.168.1.2:30008 
- -> 配置DNS服务器
- -> http://www.my-shop.com:30080 
- -> 添加代理服务器 / Proxy Server
- -> http://www.my-shop.com 
- -> Done!


**🤔 当App在云服务器上时（比如Google Cloud Platform），如何实现呢？**

- -> `LoadBalancer`类的Service，不仅拥有`NodePort`类的功能，还向云服务器发送了一个 **添加负载均衡器(`LoadBalancer`)** 的请求
- -> 云服务器自动配置一个负载均衡器（⚠️ **负载均衡器** 的价格是按“个”计算的！）
- -> 该**负载均衡器**会发布一个可供外部访问的IP（我们叫他`LoadBalancerIP`）
- -> 配置DNS服务器，指向`LoadBalancerIP`
- -> `http://www.my-shop.com`

⚠️**负载均衡器** 可嵌套，比如对一个主域名下的子域名进行配置：

![请添加图片描述](../ckad-6/946b9dbbe3d2414f95462e338985f943.png)

这些步骤都完成以后，配置`SSL`以使用`https`（而不是`http`）。对`SSL`的配置可以在多个位置实现：

- App本身层面
- 负载均衡器层面
- 代理服务器层面

上述三种配置方法，均可实现同样的功能。

## 救星Ingress
各种各样的配置，你的头现在多大了呢？Ingress就是用来帮忙解决这个令人头疼的问题的！它包括但不限于路由（包括引流到不同App，比如`/shopping` 和`/game`）和`SSL`配置。

<img src="../ckad-6/37c75713b9714222a1190665d0d83e57.png" width=800 />


## Ingress组成
把Ingress分为两个组件：**Ingress Controller**和**Ingress Resource**。一般两者分别在不同的namespace中，特别是**Ingress Controller**，做为与应用无关的资源，应该被放到单独的Namespace中！
<!-- TODO： draw this pic better: ingress-controller-space & app-space -->
<img src="../ckad-6/ingress.png" width=700 />

### (1) Ingress Controller
Ingress控制器由四个资源组成：
1. 负载均衡器（以Deployment的形式存在）
2. ConfigMap
3. NodePort类的服务
4. ServiceAccount

#### **1.一个负载均衡器**
以一个单独软件的形式deploy到当前的cluster上。较为流行的有`GCE HTTPS LoadBalancer（谷歌）`，`Nginx`，`Contour`，`HaProxy`，`taefik`和`Istio`。其中`GCE HTTPS LoadBalancer（谷歌的）`和`Nginx`由Kubernetes管理。

我们这里以`Nginx`为例：命名该Deployment为`nginx-ingree-controller`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: nginx-ingree-controller
spec:
	replicas: 1
	selector:
		matchLabels:
			name: nginx-ingress
	template:
		metadata:
			labels:
				name: nginx-ingress
		spec:
			containers:
				- name: nginx-ingress-controller
				  image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
			args:
				- /nginx-ingress-controller		# nginx程序存储在该文件夹下，所以要在运行时进入文件夹
				- --configmap=${POD_NAMESPACE}/nginx-configuration	# 使用CM存nginx设置，而不是直接写在当前的Deployment里面
			env:
				- name: POD_NAME
				  valueFrom:
				  	fieldRef: 
				  		fieldPath: metadata.name
				- name: POD_NAMESPACE
				  valueFrom:
				  	fieldRef: 
				  		fieldPath: metadata.namespace
			ports:
				- name: http
				  containerPort: 80
				- name: https
				  containerPort: 443
```

!!! note
	所使用的镜像是一个专门用来做Ingress的nginx镜像，可以在[这里](https://quay.io/repository/kubernetes-ingress-controller/nginx-ingress-controller)找到。

#### **2.一个ConfigMap**
这里命名为`nginx-configuration`，该ConfigMap被1中的Deployment使用，用来存储和修改`nginx`配置，比如存储日志的路径，SSL 设置，最长等待时间等。
```yaml
apiVersion: v1
kind: ConfigMap
metadata: 
	name: nginx-configuration
...
```

#### **3.一个NodePort类型的Service**
使Deployment `nginx-ingree-controller`可供外部访问
```yaml
apiVersion: v1
kind:  Service
metadata: 
	name: nginx-ingress-service
spec:
	type: NodePort
	ports:
		- port: 80
		  targetPort: 80
		  protocol: TCP
		  name: http
		- port: 443
		  targetPort: 443
		  protocol: TCP
		  name: https
	selectors:
		# 对应Deployment的Label
		name: nginx-ingress
```

#### **4.一个ServiceAccount**
`nginx` Ingress还提供了额外的功能，比如监控从集群外部来的访问，并实时修改nginx配置。该操作需要正确的权限，所以我们需要一个ServiceAccount，并在Deployment中使用该SA
```yaml
apiVersion: v1
kind:  ServiceAccount
metadata: 
	name: nginx-ingress-serviceaccount
# Roles, ClusterRoles, RoleBindings...
```

### (2) Ingress Resource
Ingress Resource是应用于Ingress Controller的**一组规则和配置**。比如：

- 对`www.my-shop.com/shopping`的访问流入到`Shopping App`应用中，而对`www.my-shop.com/game`的访问流入到`Game App`应用中
- 或者直接从domain就开始区分，比如常见的地域区分：`www.apple.cn`和`www.apple.de`

我们用yaml文件定义一个Ingress Resource。比如，下面这个最简单的例子，会将所有访问都导入到名为`shopping-service`的Service中去
```yaml
# ingress_resource.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress # 注意是Ingress类型
metadata:
	name: ingress-rule
spec: 
	backend:
		serviceName: shopping-service
		servicePort: 80
```

生成Ingress Resource：
```bash
kubectl create -f ingress_rule.yaml
```
查看生成的Ingress Resource：
```bash
kubectl get ingress
```
##### 设置默认backend
`k describe ingress/ingress-rule` 可以看到一个`Default backend`。这个名为`default-http-backend`的服务是当用户输入的DNS地址（即网址）和规则中任意一条都不相符的时候会被访问！所以可以手动建一个这个名字的服务，然后访问一个自定义的，更加友好的404页面	
![请添加图片描述](../ckad-6/d5c34e0f5b2444d8977839d2de1e2c0e.png)

这时查看Yaml文件，该ingress大概长这样：
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-backend
spec:
  defaultBackend:	# 默认服务
    resource:
      apiGroup: k8s.example.com
      kind: StorageBucket
      name: static-assets
  rules:					# 规则
    ...
```

##### rewrite-target


```yaml
annotations:
	# the `/` is the path that replaces the original path
	nginx.ingress.kubernetes.io/rewrite-target: /
```

!!! note "举例"
		当某个Ingress提供对多个应用的访问，比如：

		- `/frontend` -> `frontend-service` -> `frontend-react-app`
		- `/dashboard` -> `dashboard-service` -> `dashboard-power-bi-app`

		`http://example.com/frontend/home`会被重新成`http://example.com/home`。
		
		而`frontend-react-app`软件本身提供的页面是没有子域名的，也就是说，如果我在本地跑react软件，能访问到的页面是`localhost:3000`而不是`localhost:3000/frontend`，所以需要“rewrite”


#### Ingress规则
!!! info "例1：同一个域名，不同子路径"
	假设我们想要实现一个分流，所有访问`www.my-shop.com/shopping`都将访问导入到名为`shopping-service`的Service，当个用户访问`www.my-shop.com/game`时，访问另一个名为`game-service`的Service。我们可以在Ingress Resource的yaml文件中写规则（rule），举例：
	```yaml
	# my_ingress_with_path.yaml
	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
		name: ingress-rule
	spec: 
		rules:
		# 这里host是可选的，如不指定，则该规则适用于通过指定 IP 地址的所有入站 HTTP 通信
		- host: www.my-shop.com
		  http:
			paths:
			- path: /shopping
			pathType: Prefix
			backend:
				service:
					name: shopping-service
					port: 
						number: 80
			- path: /game
			pathType: Prefix
			backend:
				service:
					name: game-service
					port: 
						number: 80
	```
	此外，也可以用`k`命令直接生成Ingress：
	```bash
	kubectl create ingress ingress-rule \
		--rule="www.my-shop.com/shopping*=shopping-service:80" \
		--rule="www.my-shop.com/game*=game-service:80"
	```

!!! info "例1：不同域名"
	```yaml
	# my_ingress_with_host.yaml
	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
		name: ingress-rule
	spec: 
		rules:
		- host: www.my-shop.cn
		http:
			paths:
			backend:
				serviceName: my-shop-cn
				servicePort: 80
		- host: www.my-shop.de
		http:
			paths:
			backend:
				serviceName: my-shop-de
				servicePort: 80
	```

<!-- ![请添加图片描述](../ckad-6/1a71f7d2c32d462c8a9ae149a739bea6.png) -->

!!! warning
	Ingress默认只访问当前Namespace中的服务，但也可以有例外：

	Alternatively there is way to achieve ingress in one namespace and service in another namespace via externalName.Checkout Kubernetes Cross Namespace Ingress Network：
	```yaml
	kind: Service
	apiVersion: v1
	metadata:
		name: test-service-1
		namespace: namespace-a
	spec:
		type: ExternalName
		externalName: test-service-2.namespace-b.svc.cluster.local
		ports:
		- port: 80
	```

#### Bug & Debug
很多时候Ingress出问题是因为它的annotations中缺少了一些东西。我们可以通过查看它所指向的Pod的log，或者ingress controller的log查看具体问题出在哪儿：
```bash
# 查看Ingress所指向的某个App 的 log
k logs webapp-wear-658fc8dbb4-4m6gs -n app-space 

# 查看Ingress controller 的 log
k logs ingress-nginx-controller-ffc6b494b-b7vms -n ingress-nginx 
```

常见问题：

!!! warning "503 Service Temporarily Unavailable"
		`pathType: Exact` which should be `pathType: Prefix`

!!! warning "301 too many redirect"
		missing this:

		```yaml
		metadata:
			annotations:
		    nginx.ingress.kubernetes.io/ssl-redirect: "false"
		```

		By default the controller redirects (301) to HTTPS if TLS is enabled for that Ingress. If you want to disable that behavior, you can use the nginx.ingress.kubernetes.io/ssl-redirect: "false" annotation.

		And in the lab, we manage the HTTPS from our side, not from the ingress. So, you should add annotation nginx.ingress.kubernetes.io/ssl-redirect: "false"

!!! warning "svc didnt reach the correct one"
		missing this
		```yaml
		metadata:
			annotations:
				nginx.ingress.kubernetes.io/rewrite-target: /
		```





# 3. NetworkPolicy
<img src="../ckad-6/6ada1cda6fbf42439357ee2a4d31445b.png" width=200 />

NetworkPolicy（`netpol`） 是 Kubernetes 中用于控制 Pod 网络流量的规则。通过 Network Policies，可以定义哪些流量（入站和/或出站）可以到达 Pod 或从 Pod 发出。




!!! info "理论"
	- **概念：** ingress和egress都是 **相对** 的概念，收到的请求叫ingress，发出的叫egress
	- **背景：** 理论上一个cluster上所有的`Pod`之间都能通过`Service`互相沟通，但如图，我们可能不需要也不想要 Frontend 和 Database 之间直接进行沟通，我们可以通过 **Network Policies** 对某个`Pod`的traffic设限
	- **Network Policies**：通过`label`和`selectors`将 **Network Policies** 连接到`Pod`上去

NetworkPolicy举例：YAML中的`.spec.podSelector.matLabels.name`指定了 **目标 Pod 标签**。
```yaml
# policy-definition.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
	name: db-policy
	namespace: prod
spec:
	podSelector:
		matchLabels:
			role: db 		# 说明此NetPol用在有“role: db”标签的pod上
	policyTypes:			
		- Ingress			# 这里写明了规则是限制了Ingress还是Egress还是两者都，这里两个都限制了
		- Egress
	ingress:				# 对ingress的规则：
		- from:
			- podSelector:				# 元素1
				matchLabels:
					role: api-pod
			  namespaceSelector: 		# 可选项目：如果api-pod不在当前的namespace，则需要注明selector
			  	matchlabels:
			  		name: staging
			- ipBlock:					# 元素2：与元素1之前是“或”的关系
				cidr: 192.168.5.12/32
		  ports:
		  - protocol: TCP
		    port: 3306
	egress:					# 对egress的规则：
		- to:
		  - ipBlock:
		  		cidr: 192.168.5.12/32
		  ports:
		  - protocol: TCP
		    port: 80
```

!!! note
	无论是 ingress 还是 egress的规则，都由两部分组成：
	
	1. `ingress.from`规定了通信的来源 （`egress.to` 规定了通信的目的地）
	2. `ports`规定了使用当前Pod的哪个端口进行通信。
	
	⚠️ 如果只有`namespaceSelector`，没有`podSelector`，则表示该规则适用于某namespace上的所有Pod <br/>
	⚠️ `ipBlock`：假设规则想要应用到某个不在当前`Cluster` 上的App，则可使用它

!!! note "`cidr`"
	CIDR（Classless Inter-Domain Routing）表示法用于定义 IP 地址范围，格式为：

	```bash
	<IP地址>/<前缀长度>
	```

	举例：

	- `192.168.5.12/32`：表示 单个 IP 地址（`192.168.5.12`），因为 /32 表示网络掩码 `255.255.255.255`，即仅匹配这一个 IP。
	- `192.168.5.0/24`：表示 `192.168.5.0` 到 `192.168.5.255` 的所有 IP。
	- `10.0.0.0/8`：表示 `10.0.0.0` 到 `10.255.255.255` 的所有 IP。


创建Network Policies：

```bash
kubectl create -f policy-definition.yaml
```

!!! warning "AND vs OR"
	`from`中包含的内容是否作为数组的元素存在很重要！如下：

	| As combined item | As separated items in list |
	|--|--|
	|For example, `namespaceSelector` AND `podSelector` <br/> <img src="../ckad-6/c00298c19d8b4c8c992d5f4cc135859a.png" /> |For example, `namespaceSelector` OR `podSelector` <br/>  <img src="../ckad-6/8cae9a4ae09c4c919f8ae201159ee3be.png" /> |

!!! warning "more than one `netpol`"
	If a pod is matched by selectors in one or more `netpol`s, then the pod accepts only connections that at least one of those `netpol` allows.

!!! warning
	If a pod does not match any `netpol`, then OpenShift does NOT restrict traffic to that pod.

!!! warning "Where does `netpol` apply?"
	Network policies control only **internal traffic** from pods that do not use **host networking**.

	网络策略只适用于 使用 Kubernetes 虚拟网络的 Pod 的内部流量，即 Pod 之间通过集群网络通信的流量。对于那些配置了 主机网络（host networking） 的 Pod，由于它们直接使用主机的网络栈，因此网络策略对它们的流量 不起作用:

	- **Internal Traffic（内部流量）**：指 Pod 之间通过 Kubernetes 虚拟网络进行的通信。Pod A 通过服务（Service）访问 Pod B，例如 backend-service.default.svc.cluster.local。
	- **Host Networking（主机网络）**：指 Pod 与节点共享网络栈，不再使用 Kubernetes 的虚拟网络，此时网络策略对流量无效。即，如果 Pod 配置了 `hostNetwork: true`：
		<pre><code>
		apiVersion: v1
		kind: Pod
		metadata:
			name: host-network-example
			namespace: default
		spec:
			<span style="background-color: #ccd1f0">hostNetwork: true</span>
			containers:
			- name: nginx
				image: nginx:latest
				ports:
				- containerPort: 80
		</pre></code>



# 4. 标签&选择器（Labels & Selectors）
k8s中大量地使用了标签&选择器。比如：Service通过选择器（selector）找到拥有正确标签（labels）的Pod。
```yaml
selector:
	app: myapp
	tier: frontend
```

!!! warning
	Pod要满足`selector`中所有的标签才可以！`app==myapp` 和 `tier=frontend`两个标签都要有。下面这个Pod就不满足条件：

	```yaml
	kind: Pod
	metadata:
		name: myPod
		labels:
			tier: frontend
	```

#  >>>  本章kubectl命令整理
**IP**

- `curl ifconfig.me` 查看外部IP
- `ipconfig getifaddr en0` 查看内部IP

**服务（`Service`）**

- `k get service`
- `k create service nodeport [ServiceName] --tcp=8080:8080 `
- ```k expose deployment/[DeploymentName] \
	--name=[ServiceName] \
	--port=8080 \
	--target-port=8080 \
	--type=NodePort
	```

**Ingress**

- `k get ingress`
- `k describe ingress/ingress-rule`
- ```k create ingress ingress-rule \
		--rule="www.my-shop.com/shopping*=shopping-service:80"
		--rule="www.my-shop.com/game*=game-service:80" 
	```


**Network Policies**

- `k get networkpolicies`
- `k get netpol`
