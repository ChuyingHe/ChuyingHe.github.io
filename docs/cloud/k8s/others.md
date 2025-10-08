# EndpointSlice
`EndpointSlice` 是一个 Kubernetes API 对象，它跟踪的是：一个 Service 背后具体是哪些 Pod（它们的 IP 地址和端口）在接收流量。


1. 你创建一个 Service，并定义它的 selector（例如 `app: my-api`）。
1. Kubernetes 的控制平面（主要是 endpoint-slice 控制器）会持续监视集群中 Pod 的变化。
1. 每当有带有 app: my-api 标签的 Pod 启动并变为就绪状态时，控制器就会自动创建一个 EndpointSlice 对象，或者更新现有的一个，将这个新 Pod 的 IP 和端口添加到该 Service 对应的 EndpointSlice 中。
1. 当 Pod 被删除或终止时，控制器又会自动将其从 EndpointSlice 中移除。
1. kube-proxy 这个组件在每个节点上运行，它会监听 EndpointSlice 的变化，并据此更新节点上的 iptables 或 IPVS 规则。
1. 当流量发往 Service 的虚拟 IP 时，这些网络规则会根据负载均衡策略（如轮询）将流量直接转发到 EndpointSlice 中列出的某个真实 Pod IP 上。