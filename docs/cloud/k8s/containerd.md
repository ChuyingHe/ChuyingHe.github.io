# ctr
**ctr** is a CLI tool to interact with **containerd**. 底层调试工具，是 containerd 的“原装”但对用户不友好的螺丝刀。

- no website
- for doc, use `ctr --help`
- uses <u>containerd original API</u>
- 名字 ctr 是 **containerd** 的缩写



# crictl

**crictl** is a CLI tool to interact with **containerd**. Kubernetes 节点调试工具，用于检查 Kubernetes 创建的容器和 Pod。

- website: https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md
- uses <u>CRI (Container Runtime Interface) API</u>
- 名字是 crictl 是 **CRI control** 的缩写