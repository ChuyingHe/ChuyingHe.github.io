多Pod容器有多种**设计模式**，包括**Ambassador**，**Adapter**和**Sidecar**。我们会一一对他们进行学习。

# 1. 为何需要多容器Pod
将大型单体应用程序（monolith）解耦为称为**微服务**（microservices）的子组件的想法使我们能够开发和部署一组独立的小型可重用代码。 这种架构可以帮助我们根据需要扩大、缩小和修改每个服务，而不是修改整个应用程序。如下图：
<img src="../ckad-3/0cb1ca41162446ce8e7d2531568ec3bd.png" width=700 />

*（图片[来源](https://martinfowler.com/articles/microservices.html)）*

但是，有时您可能需要两个服务一起工作，例如 Web 服务器和日志服务。每个配对的 Web 服务器实例需要一个日志服务的实例。所以我们需要把这两个微服务放到同一个Pod中，以保证它们：

- 处于同一个生命周期（lifecycle）：意味着同时被生成/销毁
- 分享同一个network：可以相互引用为`localhost`
- 访问同一个volume

当用户访问量增多，我们需要扩大应用规模的时候，我们只需多建几个Pod就可以了：
<img src="../ckad-3/scale.png" width=800 />

举例：多容器Pod的定义文件

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp
spec:
	# containers是一个数组，数组中的每个元素是一个container
	containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
    - name: log-agent
      image: log-agent
```
#	2. 多容器Pod的设计模式
<!-- TODO: draw pictures -->

## Sidecar
在同一个`Pod`中，给每个配对的 Web 服务器配一个日志服务的实例。然后将不同Pod中日志服务记录下来的日志发送到一个**中央服务器**。

<img src="../ckad-3/pod_pattern_sidecar.png" width=600 />

## Adapter
是基于**Sidecar**设计模式的变种。唯一的区别是在发送日志到**中央服务器**之前，先对日志格式进行处理，以便**中央服务器**可以收到数据格式相同的对日志。

<img src="../ckad-3/pod_pattern_adapter.png" width=600 />


## Ambassador
假设你有`dev`，`prod`两个个数据库，分别对应开发和生产两个应用场景。如何判断当前环境，并选择正确的数据库呢？我们可以选择将此类逻辑外包给 `Pod` 中的**某一个特定容器**，该容器就扮演了一个**大使（Ambassador）**的角色。

<img src="../ckad-3/pod_pattern_ambassador.png" width=600 />


!!! warning
    这三种是不同的软件设计模式，当我们用yaml文件写`Pod`时，定义文件其实并 **没有** 区别！

# 3. initContainer
我们之前提及的`Pod`中的`Container`都是在`Pod`起来之后就一直在运行的。`Pod`生`Container`生，`Pod`死`Container`死，在同一个生命周期。

但还有一种应用场景，`Container`上的程序只需要跑一次，不需要整个生命周期都在执行。比如，把源代码从github上下载下来 -- 这件事情就只需要做一次。`initContainer`就是解决办法。

对`initContainer`的定义和普通`Container`是类似的：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo Hello Word! && sleep 3600']
  
  initContainers:   # 和containers一样，initContainers也放在一个数组里
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <my-repository> ;']
```

**initContainer 运行规则：**

1. 如果有一个 `initContainer` 未能运行完成，则Kubernetes 会反复重启 Pod，直到`initContainer` 运行成功为止
2. 如有多个`initContainer`，则会按序一个一个运行
3. POD首次创建时，会运行 `initContainer`，如果`initContainer`和`Container`同时存在，`Container`要在所有的`initContainer`跑完之后才跑！


# 4. Debug

- `k describe pod/xxx`是个非常强大的命令。比如能看到状态，原因，错误代码等。
<img src="../ckad-3/58a739a5ab3d4fdc96fd626f24355397.png" width=900 />
- `k logs pod/xxx` 打印该pod的日志
- `k debug pod/xxx` 和`k exec`一样， 进到Pod里面去，执行它的bash。区别是，`k debug`能在容器有错的情况下，强制启动并进行debug，而`k exec`只能在Pod是处于运行状态下才能执行

#  >>>  本章kubectl命令整理
-- -- 
**YAML语法**

```yaml
command:
    - sh
    - -c
    - sleep 
    - "20"
```
⚠️ 纯数字要加双引号

**JSON语法**：

```yaml
command: ["sleep", "20"]
```
-- -- 
**修改Pod**

除了 “（1）输出yaml（2）删除旧Pod（3）从yaml文件新建Pod” 的方法之外，还有一种更简单的方法：

- `k edit pod/xxx`这里你会得到提示信息说Pod无法被直接修改，你的改变被暂时存到了`/tmp/kubectl-edit-xxxx.yaml`文件中
<!-- TODO: screenshot -->
- `k replace --force -f /tmp/kubectl-edit-xxxx.yaml`强制用新的YAML文件代替旧的Pod
