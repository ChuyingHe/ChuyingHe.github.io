# Air-gapped VS Restricted Network

|对比项|Air-gapped Network|Restricted Network|With Network Customization|
|:-|:-|:-|:-|
|是否能访问互联网|❌ 完全不能|⚠ 受限，可通过 Proxy 或特定白名单|可以访问互联网	|
|需要 Proxy Server|否|必须|可选|
|是否必须用本地 Registry|是，<br/>需要手动下载并导入|否，<br/>可能需要 Proxy 访问镜像仓库|否，<br/>从网络下载，无需本地registry|
|适用场景|高安全性、军事、政府、企业内网|受控企业网络、私有云环境|公有云、混合云、企业定制 SDN|

## 在 OpenShift 部署中的影响

**Air-gapped 部署：**

- 需要 本地 Registry（镜像仓库），如 镜像镜像 (mirror registry)。
- 需手动下载 OpenShift 安装包、Operator、更新包，并手动导入。
- 无法在线拉取容器镜像，所有组件必须本地化。

**Restricted Network 部署：**

- 可能需要配置 http_proxy、https_proxy 让集群访问特定的外部资源。
- 可以设置防火墙规则，允许访问 Red Hat 镜像仓库（如 quay.io）。
- 仍然可以使用 OpenShift Disconnected Registry，以减少对外部访问的依赖。
