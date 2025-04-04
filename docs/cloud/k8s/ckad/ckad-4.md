# 1. Pod的生命周期
被用来形容`Pod`的状态的有**Pod Status**和**Pod Condition**，总的来说 Status 是整体状态的高层次总结，Condition 提供更细粒度的信息，描述 Pod 的具体运行条件和状态。

|特性|Pod Status|Pod Condition|
|:-|:-|:-|
|目的|提供 Pod 的整体生命周期状态。|提供 Pod 的详细运行条件信息。可以看到带timestamp的历史条件变化|
|字段位置|`.status.phase`|`.status.conditions`|
|可能值|只有有限的几个可能值：<br/>Pending, Running, Succeeded, Failed, Unknown|是一个数组，每个条件都有类型、状态、最后变化时间等字段，可能的条件有：<br/>PodScheduled, Ready, Initialized, ContainersReady 等。|
|详细程度|高层次、简要总结。|细粒度、多条件描述。|
|信息用途|查看 Pod 的生命周期状态，便于快速判断当前状态。|查看 Pod 的具体状态变化，便于排查问题或调试。|


## Pod Status 
|Pod Status|解释|
|:--|:--|
|`Pending`|`Pod`刚刚建成， **k8s的Scheduler** 正在考虑把`Pod`放在哪一个`Node`上。如果Scheduler没有办法决定把`Pod`放哪里那么就会一直处于`Pending`状态<br/><br/>可用 `k describe pod/xxx` 查看原因（比如是资源不足导致的）|
|`ContainerCreating`|这是`Pod`已经被Scheduler顺利分配到`Node`中了。接下来会下载镜像并创建容器|
|`Waiting`|Pod被排到了某个`Node`上，但出于某种原因，`Pod`没有办法跑。原因多种多样，比如镜像没有办法下载等|
|`Running`|容器顺利地创建好了，App在跑了|


!!! note "Example in YAML file"
    ```yaml
    status:
      phase: Running
    ```
    
!!! note "k describe"
    `k describe pod/xxx用于`查看`Status`属性：

    ```bash
    NAME       READY   STATUS      RESTARTS      AGE
    elephant   0/1     OOMKilled   3 (33s ago)   54s
    monkey     1/1     READY   	   3 (33s ago)   54s
    ```
    ⚠️ 结果中`READY`列代表：`Pod中READY的Container的数量` / `Pod中Container的总数`，比如`1/1`

## Pod Condition

|Pod Status|解释|
|:--|:--|
|`PodScheduled`|`Pod`被Scheduler顺利分配到`Node`中|
|`Initialized`|所有 InitContainer 都成功执行完毕|
|`ContainersReady`|`Pod`上所有容器都运行正常|
|`Ready`|`Pod`上所有容器可以通过Service被用户访问到 --> 这个属性也能在`k get pods`中的表格中看到，例子如下：|


!!! note "Example in YAML file"
    ```yaml
    status:
      conditions:
        - type: PodScheduled
          status: True
          lastProbeTime: null
          lastTransitionTime: "2024-12-01T10:00:00Z"
        - type: Ready
          status: True
          lastProbeTime: null
          lastTransitionTime: "2024-12-01T10:05:00Z"
        - type: ContainersReady
          status: True
          lastProbeTime: null
          lastTransitionTime: "2024-12-01T10:05:00Z"
        - type: Initialized
          status: True
          lastProbeTime: null
          lastTransitionTime: "2024-12-01T09:55:00Z"
    ```

# 2. Readiness和Liveness监测
## Probe的三种写法
Pod Condition中的`Ready`不能保证Container中的App已经可以接受用户访问了。比如Jenkins服务器刚刚开始跑的时候需要大改10-15秒时间，所以在这10-15秒中内。`Pod`会告诉我们Condition是`Ready`了，但其实App本身并没有起来。我们给Container添加 Probes，用 Probes 来测试App是否成功运行。**Probes**有三种写法：

1. **HTTP测试**: for web applications
```yaml
httpGet: 
   path: /api/ready
   port: 8080
```
2. **TCP socket测试**: for services that don't expose HTTP but should be reachable via a port.
```yaml
tcpSocket: 
   port: 3306
```
3. 自定义的 **命令脚本**: for checking internal states or system-level conditions.
```yaml
exec: 
   command:
   	- cat
   	- /app/is_ready
```

### TCP vs HTTP

<video
  muted
  loop
  preload="auto"
  autoPlay
  playsInline
  src="../ckad-4/tcp_vs_http.mp4"
  width="700"
></video>



**Probe的配置**

- `initialDelaySeconds: 10`：第一次用Probe之前先等10秒（单位：Seconds）
- `periodSeconds: 5`：每5秒用一次Probe（单位：Seconds）
- `failureThreshold: 8`：最多尝试8次，如果第八次还是失败，那就停止尝试并设`Ready`为FALSE

## Probe的使用
### ReadinessProbe
**ReadinessProbe**给Pod Condition的`Ready`加了一个门槛：只有当ReadinessProbe中的测试得到正确结果后才能给Pod一个`Ready`的Condition。举例：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080

    readinessProbe:           # 预备测试
      initialDelaySeconds: 10 # 更多复杂设置
      periodSeconds: 5
      failureThreshold: 8
      httpGet:                # httpGet测试
        path: /api/ready
        port: 8080
```
**Probe**保证了标记为`Ready`的容器是正常运行的。保证了软件更新时用户总是能成功访问到我们的软件。没有`Ready`之前，所有的traffic都会被导到旧的`Pod`上去，等新的`Pod`确定`Ready`了之后，才会对它进行使用。

### LivenessProbe
!!! note "Docker VS Kubernetes"
    当我们用Docker创建一个`Container`时，如果一个`Container`突然出问题，其中的App无法被使用了，那需要开发者手动删除出问题的`Container`，再重新建一个。而Kubernetes会自动尝试删除旧的`Container`，建一个新的 --> Pod/Container由Deployment管理和执行

有时会我们会遇到`Container`本身没有问题，但是`Container`中的App因为某个bug而出错的情况（比如，python代码中某个依赖包无法找到）。这种情况下`Pod`没有办法监测到内部App的问题，我们就需要用到**LivenessProbe** ，它可以定期测试容器内的应用程序是否真的健康。如果测试失败，则该`Container`被认为不健康，会被销毁并重建。举例：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080

    livenessProbe:        # livenessProbe
      httpGet: 
        path: /api/ready
        port: 8080
```

# 3. 容器日志（Logging）
**Docker**中，我们可以用 **非分离** 的模式创建容器xxx：

```bash
docker run [ImageName] --name [ContainerName]
```
该模式下，如果用Ctrl+C退出，该容器会被暂停运行。如果我们想要容器一直在背景中运行，我们可以用 **分离模式** 跑容器（`d`指`detached`，即”分离“）：
```bash
docker run -d [ImageName] --name [ContainerName]
```
此时如果我们想要查看容器中的日志，可以用以下命令查看，`-f`表示`follow`打印实时日志：
```bash
docker logs -f [ContainerName] 
```


同样的，在**k8s**中，我们也可打印实时日志:
```bash
kubectl logs -f [PodName]   # ⚠️ 在[PodName]前面不加 `pod`
```

当一个`Pod`中包含 **多个** `Container`时，我们需要写明打印哪一个`Container`的日志：`-c`代表`container`
```bash
kubectl logs -f [PodName] -c [ContainerName]
```

# 4. 监测和Debug
根据应用场景，我们可能会想要监测`Node`层面和`Pod`层面的指标：

- `Node`层面：
	- 集群上`Node`的总数
	- 健康的`Node`数量
	- CPU消耗
	- 内存消耗
	- 磁盘利用率（disk utilization）
- `Pod`层面：
	- `Pod`数量
	- 每个`Pod`的CPU，内存消耗

k8s没有提供自带的监测工具，但是有很多开源的第三方服务，比如`MetricsServer`，`Prometheus`， `Elastic Stack`，`DataDog`，`Dynatrace`等。在CKAD的考试中，我们只需了解如何使用`MetricsServer`即可，其他的工具会在CKA中讲到。

## MetricsServer
`MetricsServer`的前身是`Heapster`（已弃用）。每个k8s集群有一个`MetricsServer`。`MetricsServer`收集Node和Pod的日志，并保存到 **内存** 中（不在磁盘上！），所以没有办法查看历史日志。

### 日志的生成
每个`Node`上都有一个**kubelet**代理，负责接收Master的指令，并且管理该`Node`上的`Pod`。该代理拥有一个名为**cAdvisor**（container advisor）的子组件，专门负责收集Pod的日志。

<img src="../ckad-4/metricsserver.png" width=500 />

### MetricsServer的安装
使用minikube安装： 
```bash
minikube addons enable metrics-server
```
其他环境下的安装：
```bash
git clone https://github.com:kubernetes-sigs/metrics-server.git   # 从git下载Metric-Server的YAML文件

cd metrics-server     # 会生成一堆MetricsServer所需的资源：Pod，Service和Roles
kubectl create -f .   

kubectl top node      # 等待deploy完成后，我们可以查看Node或Pod的日志
kubectl top pod
```

# 5. 常见k8s错误

|错误码| 解释 |
|:-|:-|
|`OOMKilled`| 当容器**内存不足**并且内核被迫终止进程时，会发生此错误。 OOM（Out Of Memory/内存不足）是 Linux 内核的一项功能，可以终止进程以释放内存。|
|`ImagePullBackOff`| 当 Kubernetes 无法从指定的 registry 中拉取容器镜像时会出现此错误。 这可能是由于多种原因造成的，例如**登录信息不正确**、**网络问题**或**图像名称错误**等。|
|`CrashLoopBackOff`| 当容器不断崩溃且 Kubernetes 无法成功重启时，会出现此错误。 这可能是由于**配置不正确**、**资源不足**或**应用程序代码错误**等各种原因造成的。|
|`ErrImagePull`| 当 Kubernetes 由于**身份验证问题**无法从指定的注册表中拉取容器镜像时，会出现此错误。|
|`InsufficientCPU/InsufficientMemory`| 当 `Pod` 或`Container`请求的 CPU 或内存多于节点上可用的内存时，会发生这些错误。 这可能是由于**配置不正确**或**资源不足**造成的。|
|`NodeNotReady`| 当节点由于**网络问题、资源不足**或**节点维护**等各种原因尚未准备好接受 `Pod` 时，会出现此错误。|
|`InvalidImageName`| 当**容器镜像名称不正确或无效**时会出现此错误。|
|`NotFound`| 当 Kubernetes **无法找到指定的资源**（如 pod、服务或部署）时，会出现此错误。|
|`Forbidden`| 当用户**没有访问或修改资源所需的权限**时会发生此错误。|
|`Timeout`| 当**进程完成时间过长**，并超过配置的超时期限时，会发生此错误。 这可能是由于各种原因造成的，例如**网络速度慢**或**资源不足**。|


#  >>>  本章kubectl命令整理
**生命周期**

`k describe pod/xxx`和`k get pod/xxx`都可查看Pod状态

-- --
**docker**

`docker run [ImageName] --name [ContainerName]`

`docker run -d [ImageName] --name [ContainerName]` detached 模式

`docker logs -f [ContainerName]` 打印日志

-- --

**k8s日志**

`k logs -f [PodName]`

`k logs -f [PodName] -c [ContainerName]`

-- --
**MetricsServer安装**

用minikube安装：`minikube addons enable metrics-server`

其他环境安装（直接下源码）：
```bash
git clone https://github.com:kubernetes-sigs/metrics-server.git
cd metrics-server
kubectl create -f .
kubectl top node
kubectl top pod
```
