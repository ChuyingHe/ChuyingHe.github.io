# 1. LoadBalancer Services
!!! note "**k8s Ingress** / **OC Route**"
    Both **k8s Ingress** and **OC Route** are mainly designed to `HTTP/HTTPS` services. To expose _non-HTTP services_, you use **LoadBalancer service**

k8s provides different `service`（different service types check [here](/cloud/k8s/ckad/ckad-6/#_1)）. `Loadbalancer` services require the use of network features that are not available in all environments.

For example, **cloud providers**(such as IBM Cloud or G-cloud) typically provide their own load balancer services. These services use features that are specific to the cloud provider.

If you run a Kubernetes cluster on a cloud provider, controllers in Kubernetes use the cloud provider's APIs to configure the required cloud provider resources for a load balancing service. On environments where managed load balancer services are not available, you must configure a **load balancer component** according to the specifics of your network - for example, **MetalLB**:

## MetalLB
**MetalLB** 是一个开源的负载均衡器解决方案(**load balancer component**)，专为<ins>没有内置云负载均衡器的</ins>集群设计。比如**bare metal cluster**, **clusters on hypervisors(虚拟机管理程序)** 或私有云. 

**MetalLB** is an **Operator** that you can install per **Operator Lifecycle Manager**:

<img src="../imgs/metallb.png" />

!!! info "step-by-step"
    1. Deploy MetalLB: Ensure MetalLB is installed and running in the cluster.
    2. Create the `IPAddressPool` resource
    3. Configure MetalLB Advertisement: create either `L2Advertisement` or `BGPAdvertisement` resource
    4. Create a LoadBalancer `service` and MetalLB will allocate IPs from the specified pool.

Example: create `service` of the LoadBalancer type to expose non-HTTP services outside the cluster.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-lb
  namespace: example
spec:
  ports:
  - port: 1234 
    protocol: TCP
    targetPort: 1234
  selector:
    name: example
  type: LoadBalancer
```
After you create the service, the **load balancer component**(such as **MetalLB**) updates the service resource with information such as the public IP address where the service is available. You can get the IP address by:

<pre><code>
[user@host ~]$ kubectl get service
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
example-lb   LoadBalancer   172.30.21.79     <span  style="background-color: #FFFF00">192.168.50.20   1234</span>:31265/TCP   4m7s

<!-- or -->
[user@host ~]$ oc get example-lb -o jsonpath="{.status.loadBalancer.ingress}"
[{"ip":"192.168.50.20"}]
</code></pre>

You can now connect to the `service` on port 1234 of the 192.168.50.20 address. You can test it in several ways:
```bash
ping 192.168.50.20:1234
# or
nc -vz 192.168.50.20 1234
```


!!! note "MetalLB's 两种模式"

    **1. Layer 2 模式**：使用 ARP/NDP 通告，将服务 IP 广播给同一子网内的所有主机。这种模式适用于简单网络拓扑。
    **2. BGP（Border Gateway Protocol）模式**：与网络中的路由器对等，动态宣告服务 IP。这种模式适合更复杂的网络环境，支持更高级的流量路由和负载均衡。

!!! info "Configure MetalLB Advertisement"
    You can use `IPAddressPools` resource for further configuration, for instance, to restrict the available load balancer IP addresses to 192.168.50.20 and 192.168.50.21. Example:
    ```yaml
    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
    name: my-ip-pool
    namespace: metallb-system  # Ensure this is the namespace where MetalLB is deployed
    spec:
    addresses:
        - 192.168.50.20-192.168.50.21  # The range of IP addresses for the pool
    ```

    ⚠️ To bind the `IPAddressPool` to MetalLB's AddressPool settings, you need to configure a `L2Advertisement` or `BGPAdvertisement`

# 2. Multus Secondary Networks
Kubernetes manages a **pod network** and a **service network**:
- The **pod network** provides network interfaces to each pod, and by default, provides network communication between all pods. 
- The **service network** provides stable addressing for services that run on pods. Furthermore, other facilities provide mechanisms to expose services outside the cluster.


However, in some cases, connecting some pods to a custom network could be useful - **The Multus CNI (container network interface)** plug-in helps to attach pods to custom networks. The custom networks could be internal OR external.

## Multus 安装
[Multus Doc](https://github.com/k8snetworkplumbingwg/multus-cni)

**Multus CNI 插件** 允许集群中的 Pod 使用多个网络接口。它充当一个多功能的网络插件，能够将多个 CNI 插件（如 Flannel、Calico、SR-IOV 等）整合在一起，赋予每个 Pod 连接到多个网络的能力。

--> This is accomplished by Multus acting as a **"meta-plugin"**, a CNI plugin that can call multiple other CNI plugins.


!!! warning "Multus vs Default CNI in cluster"
    - The **default CNI plugin **(like `Calico` or `Flannel`) continues to function as the primary network provider for the cluster, handling the standard networking features such as Pod-to-Pod communication and IP management.
        - k8s usually uses `Calico` or `Flannel` as default
        - oc uses `SDN` or `OVN-Kubernetes`
    - **Multus meta-plugin** adds support for secondary networks by allowing Pods to connect to additional networks, but it doesn't replace the default network functionality.


!!! note "Multus CNI & other CNI"
    其他CNI 插件可通过Operators来安装。 比如 **Kubernetes NMState operator** 或 **SR-IOV (Single Root I/O Virtualization) network operator**。他们与Multus CNI的关系如图:

    <img src="../imgs/multus_cni.png" width="300" />

## 配置 Secondary Network
Two ways to configure secondary networks:

1. create a `NetworkAttachmentDefinition` resource. 
2. update the configuration of the **cluster network operator**

### 1. create `NetworkAttachmentDefinition`
<pre>
<code>
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: example <span  style="background-color: #FFFF00"># 1</span>
spec:
  config: |-
    {
      "cniVersion": "0.3.1",
      "name": "example", <span  style="background-color: #FFFF00"># 2</span>
      "type": "host-device", <span  style="background-color: #FFFF00"># 3</span>
      "device": "ens4",
      "ipam": { <span  style="background-color: #FFFF00"># 4</span>
        "type": "dhcp"
      }
    }
</code>
</pre>

!!! note "解释"
    1. The network name
    2. The same value as the name
    3. The **Network Type**
    4. Additional network configuration

!!! note "Network Type"
    You can create network attachment definitions of the following types:
    
    - **Host device**: Attaches a network interface to a single pod.
    - **Bridge**: Uses an existing bridge interface on the node, or configures a new bridge interface. The pods that are attached to this network can communicate with each other through the bridge, and to any other networks that are attached to the bridge.
    - **IPVLAN**: Creates an IPVLAN-based network that is attached to a network interface.
    - **MACVLAN**: Creates an MACVLAN-based network that is attached to a network interface.


### 2. update Network Operator

## 使用Secondary Network
Network attachment resources are **namespaced**, and are available only to pods in their namespace.

To use the **Secondary Network**, simply add `annotation` to the `deployment`:
<pre>
<code>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example
  namespace: example
spec:
  selector:
    matchLabels:
      app: example
      name: example
  template:
    metadata:
      annotations:
        <span  style="background-color: #FFFF00">k8s.v1.cni.cncf.io/networks: example</span>
      labels:
        app: example
        name: example
    spec:
...
</code>
</pre>
