
以下是Chapter 2. Converting a connected cluster to a disconnected cluster 的内容

https://docs.redhat.com/en/documentation/openshift_container_platform/4.17/html-single/disconnected_environments/index#connected-to-disconnected


 ## 2. Configure Mirror Registry
Adding the Mirror Registry certificates to the list of trusted CAs on your host.
```bash
cp </path/to/cert.crt> /usr/share/pki/ca-trust-source/anchors/

# Update the CA trust. For example, in Linux:
update-ca-trust
```
## 3. Grant Access to Registry on Internet
Creating a `.dockerconfigjson` file that contains your **image pull secret**, which is from the cloud.openshift.com token.

## 4. 拷贝 核心镜像
Copy 3 types of INDISPENSABLE images from **your external repositories on Internet** to the **Mirror Registry.**

### 4.1 拷贝 Operator Lifecycle Manager (OLM) images
OLM 负责在 OpenShift 集群中 安装、更新、管理 Operators。比较特殊，所以需要单独同步，因为它是所有 Operator 的管理层，必须先安装 OLM 后，才能安装具体的 Operators。
```bash
oc adm catalog mirror registry.redhat.io/redhat/redhat-operator-index:v{product-version} <mirror_registry>:<port>/olm -a <reg_creds>
# example: oc adm catalog mirror registry.redhat.io/redhat/redhat-operator-index:v4.8 mirror.registry.com:443/olm -a ./.dockerconfigjson  --index-filter-by-os='.*'
```


### 4.2 拷贝其他 Red Hat-provided Operator
例如 etcd、Prometheus、Logging 等 Operators。这个命令会拉取 所有可用的 Operator，如果只需要部分 Operators，需要自定义 `catalogSource`.
```bash
oc adm catalog mirror <index_image> <mirror_registry>:<port>/<namespace> -a <reg_creds>
# example: oc adm catalog mirror registry.redhat.io/redhat/community-operator-index:v4.8 mirror.registry.com:443/olm -a ./.dockerconfigjson  --index-filter-by-os='.*'
```

### 4.3 拷贝 the OpenShift Container Platform image repository:
镜像 核心 OpenShift 组件（如 Kubernetes、etcd、SDN、控制平面等）。
```bash
oc adm release mirror -a .dockerconfigjson --from=quay.io/openshift-release-dev/ocp-release:v<product-version>-<architecture> --to=<local_registry>/<local_repository> --to-release-image=<local_registry>/<local_repository>:v<product-version>-<architecture>
# example: oc adm release mirror -a .dockerconfigjson --from=quay.io/openshift-release-dev/ocp-release:4.8.15-x86_64 --to=mirror.registry.com:443/ocp/release --to-release-image=mirror.registry.com:443/ocp/release:4.8.15-x86_64
```

## 5. 拷贝 所需的镜像
除了 OpenShift 官方组件，有些企业可能还需要 额外的容器镜像，例如 Nginx、PostgreSQL之类的：
```bash
oc image mirror <online_registry>/my/image:latest <mirror_registry>
``` 