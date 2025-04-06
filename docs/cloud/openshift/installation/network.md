# 概念
## 节点（Node）网络
> 主机=Machine=VM=Host

指给Bastion Host，Bootstrap，Masters和Workers每个VM分配的IP - 为物理/虚拟机节点分配 IP。 有两种选择：

1. 通过 DHCP（dhcpd.conf） 
2. Static IP


节点 IP 仅用于主机间底层通信（如 kubelet 与 API Server 通信）。


## Pod 网络 （clusterNetworks）
集群内 Pod 的虚拟网络（通过 SDN 如 OpenShift SDN 或 OVN-Kubernetes 实现）- 由 CNI 插件管理。


## Service 网络（serviceNetwork）
ClusterIP 类型 Service 的虚拟 IP 范围。	- 由 Kubernetes/Openshift 控制平面分配


!!! danger
    唯一要求：三者（节点、Pod、Service）**必须无冲突**! 如果节点网络和 Pod/Service 网络重叠，会导致路由混乱，集群无法通信。



# install-config.yaml
```yaml
apiVersion: v1
baseDomain: example.com
metadata:
  name: test
networking:
  clusterNetwork:                 # 虚拟私有 IP，由 Cluster 分配给 Pod
    - cidr: 10.128.0.0/14         
      hostPrefix: 23
  networkType: OVNKubernetes
  serviceNetwork:                 # 虚拟私有 IP，由 Cluster 分配给 Service
    - 172.30.0.0/16
```


!!! note "clusterNetwork"
    - `clusterNetwork`是 Pod 的网络范围
    - `clusterNetwork` 分配的IP，如果使用 Headless Service 来暴露的话，可以直接被用户从外部访问到
    - Pod 与 Pod 通信（无论是否在同一 Node 上）都走这个网段。
    
    例子中：

    - 子网是`/14`。即 Pod 的地址范围是 `10.128.0.0 - 10.131.255.255`
    - `cidr`是所有的Pod可用的一个大范围，但每个Node上其实一般不需要那么多IP，所以我们用`hostPrefix`缩小IP数量
    - `hostPrefix` 定义了 每个 Node 上 Pod 地址范围: 子网是 `/23`. 每个 Node 包含 2^(32-23) = 512 个 IP 地址（其中 510 个可用于 Pod）


!!! note "TODO: clusterNetwork vs serviceNetwork"
    ✅ 1. Pod IP 是实际存在的：
    
    - 是 网络插件（如 OVN） 通过 Overlay 网络分配的。
    - 每个 Pod 启动时就获得，网络层能直接路由到。
    - 是 “真实的终点”。
    
    ✅ 2. Service IP 是虚拟存在的：
    
    - 是 Kubernetes 控制面通过 kube-proxy 或 OVN 的 LoadBalancer 机制虚拟出来的。
    - 没有真实网卡或容器绑定这个 IP。
    - 相当于 DNS 负载均衡入口，用来转发到 Pod。

<img src="../imgs/install-config-networking.png" width=400 />


!!! danger
    When installing OpenShift, its important to make sure in the current network setting, NONE of the IP ranges that you defined in `clusterNetwork` and `serviceNetwork` are already occupied