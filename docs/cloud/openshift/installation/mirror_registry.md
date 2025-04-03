- [Doc: Mirroring in disconnected environments](https://docs.redhat.com/en/documentation/openshift_container_platform/4.17/html/disconnected_environments/mirroring-in-disconnected-environments#installing-mirroring-disconnected-about)
- [Blogs: Introducing Mirror Registry for Red Hat OpenShift](https://www.redhat.com/en/blog/introducing-mirror-registry-for-red-hat-openshift)

!!! warning
    也有可能出现有Proxy配置，但是没有 Mirror Registry 的情况。
    
    在某些受限网络环境下，集群可以通过代理服务器（http_proxy/https_proxy）直接访问 外部仓库（例如 registry.redhat.io 或 quay.io），而不需要本地镜像仓库（mirror registry）。


# 两种 mirroring 类型

> Mirror Registry  其实就是一个用来放 Images 的 App！

## 1. Connected Mirroring（连接模式镜像）
适用于 受限网络（Restricted Network），但仍然可以通过 代理（proxy） 访问外部镜像仓库（如 quay.io、registry.redhat.io）。

```txt
Internet <--> [Optional] Proxy <--> A random Machine <--> Registry
```

工作原理：

- 你的本地镜像仓库（Mirror Registry）作为**缓存**，存储常用的 OpenShift 组件镜像。
- 如果本地仓库缺少某个镜像，集群仍然可以通过代理访问官方仓库拉取。
- 定期自动同步本地仓库与 `registry.redhat.io`，以获取最新更新。

## 2. Disconnected Mirroring（离线模式镜像）

>  Some time ppl also call it **"Local Proxy Registry"**

```txt
Internet <--> A random Machine | (手动下载 & 物理导入) | Registry (as Local File)
```

工作原理：

- 你需要在一台能访问互联网的机器上，手动下载 OpenShift 所需的所有镜像（使用 oc adm release mirror 命令）。
- 然后把这些镜像 手动传输 到内部的私有 Registry。
- 集群所有节点只能访问本地镜像仓库，不能访问 quay.io 或 registry.redhat.io。

1. create Mirror registry
2. use the Mirror registry, there are 2 ways:
    - old tool: `oc adm command`
    - new tool `oc-mirror` plugin


|| Connected Mirroring|Disconnected Mirroring|
|:-|:-|:-|
|适用场景|Restricted Network:<br/>- 企业受限网络，可以访问外网但需要降低对外网依赖。 <br/>- 私有云环境，想要加速 OpenShift 镜像拉取，减少外部流量消耗。  |Airgap: <br/>- 政府、银行、军事、企业封闭环境。<br/>- 远程站点（如海上钻井、卫星基站等），没有稳定的网络连接。|

# Concepts Understanding
mirror registry - 镜像仓库 是一个存储 images 的仓库，因为我们没有办法直接访问 在网络上的仓库


# 1. 生成 Mirror Registry
!!! info "Prerequisites"
    - OC subscription
    - Host OS: RHEL8,9
    - Podman 3.4.2 or later
    - OpenSSL
    - DNS to Redhat Quary
    - vCPU: 2+
    - RAM: 8GB+
    - Storage: 1TB+

It should be a registry that supports `Docker v2-2`, for instances:

- [Mirror registry for Red Hat OpenShift](https://console.redhat.com/openshift/downloads#tool-mirror-registry)
    - a small-scale version of Red Hat Quay
    - Prerequisites of `mirror-registry` CLI tool
- Red Hat Quay ??
- JFrog Artifactory
- Sonatype Nexus Repository
- Harbor


## Mirror registry for Red Hat OpenShift
!!! note "Choice of Local Mirror Registry"

我们这里用 Mirror registry for Red Hat OpenShift 为例：

- registry runs on port `443`
- 重要文件夹：`$HOME/quay-install`
- 初试用户 `init` + 自动生成密码

步骤：

1. Install depenedencies
    ```bash
    # check
    which podman tar jq openssl

    # install in RHEL/CentOS
    yum install -y podman tar jq openssl  
    ```

1. [下载 `mirror-registry.tar.gz` 包](https://console.redhat.com/openshift/downloads#tool-mirror-registry)

1. 配置：
    ```bash
    tar -xzf mirror-registry-amd64.tar.gz

    ./mirror-registry install \
        --quayHostname <host_example_com> \
        --quayRoot <example_directory_name>
    
    # ./mirror-registry install --quayHostname $(hostname -f)
    ```

    - registry hostname: system’s hostname in long form 
    - self-signed TLS certificates
    - Container images & configuration data(including `rootCA.key`, `rootCA.pem`, and `rootCA.srl certificates`) saved at `$HOME/quay-install`
    - Credentials for initial users will be auto-generated and reported when the install completes.

1. 登陆 mirror registry
    ```bash
    podman login -u init \
        -p <password> \
        <host_example_com>:8443> \
        --tls-verify=false
    ```
1. run `--tls-verify=false` or "configuring your system to trust the generated rootCA certificates"??

### Known Bugs
!!! warning "An error occurred: exit status 127"
    dependencies missing, check:
    ```bash
    # check
    which podman tar jq openssl

    # install in RHEL/CentOS
    yum install -y podman tar jq openssl  
    ```

!!! warning "TASK [mirror_appliance : Waiting up to 3 minutes"
    ```bash
    ..."stderr": "Unit quay-app.service could not be found."...
    ```

    Check with journalctl:
    ```bash
    journalctl -u quay-app.service --no-pager --full
    ```

    Error msg:
    ```bash
    Redis                  | Could not connect to Redis with values provided in BUILDLOGS_REDIS. Error: WRONGPASS invalid username-password pair or user is disabled.   | 🔴     |
    ```
    Check:
    ```bash
    TASK [mirror_appliance : Create Redis Password Secret] **************************************************************************************************************
    ok: [USERNAME@HOSTNAME]   <<<< HERE, check if this task was executed or not. If "ok" was shown, it means not executing this task.
    ```

    Solution: https://access.redhat.com/solutions/7090795
    ```bash
    podman secret ls
    podman secret rm redis_pass
    ```

## Red Hat Quay

## JFrog Artifactory

## Sonatype Nexus Repository

```bash
docker run -d --name nexus \
  -p 8081:8081 -p 5000:5000 \
  sonatype/nexus3


```

## Harbor

## Docker
> - use official [registry image](https://hub.docker.com/_/registry) from Docker
> - Docu check [here](https://www.docker.com/blog/how-to-use-your-own-registry-2/)


```bash
colima start
docker run -d -p 5000:5000 --name registry registry:latest

# observe the logs in another terminal window
docker logs -f registry

# pull, tag and push the Image to my local registry
docker pull ubuntu
docker tag ubuntu localhost:5000/ubuntu
docker push localhost:5000/ubuntu

# remove the copy (this is not in my local registry)
docker rmi localhost:5000/ubuntu
docker images

# Validation: try to pull image from the registry
docker pull localhost:5000/ubuntu
docker images

# verifying that your local registry is reachable
curl -v http://localhost:5000/v2/
```



# 2. Mirroring OpenShift images
Mirroring the **OpenShift Container Platform image** repository

## 方法一： 用`oc adm`配置
> 即用 oc CLI。可用系统：Linux，RHEL8，9, MacOS, Windows



### 1. 拷贝
```bash
OCP_RELEASE=<release_version>
LOCAL_REGISTRY='<local_registry_host_name>:<local_registry_host_port>'
LOCAL_REPOSITORY='<local_repository_name>'
PRODUCT_REPO='openshift-release-dev'
LOCAL_SECRET_JSON='<path_to_pull_secret>'
RELEASE_NAME="ocp-release"
ARCHITECTURE=<cluster_architecture>
REMOVABLE_MEDIA_PATH=<path>

oc adm release mirror -a ${LOCAL_SECRET_JSON}  \
     --from=quay.io/${PRODUCT_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} \
     --to=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY} \
     --to-release-image=${LOCAL_REGISTRY}/${LOCAL_REPOSITORY}:${OCP_RELEASE}-${ARCHITECTURE} \
     --print-mirror-instructions=idms
```
⚠️ Record the entire imageContentSources section from the output!

### 2. 拷贝 Cluster Samples Operator
### 3. 拷贝 Mirroring Operator catalogs

restricted: https://docs.redhat.com/en/documentation/openshift_container_platform/4.17/html/disconnected_environments/mirroring-in-disconnected-environments#olm-mirror-catalog-colocated_installing-mirroring-installation-images

airgapped: https://docs.redhat.com/en/documentation/openshift_container_platform/4.17/html/disconnected_environments/mirroring-in-disconnected-environments#olm-mirror-catalog-airgapped_installing-mirroring-installation-images


## 方法二： 用`oc mirror` v2 配置
> - 可用系统：RHEL8，9
> - [Doc](https://docs.redhat.com/en/documentation/openshift_container_platform/4.17/html/disconnected_environments/mirroring-in-disconnected-environments#about-installing-oc-mirror-v2)

### 1. 下载oc-mirror.tar.gz

```bash
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/latest/oc-mirror.rhel9.tar.gz
tar -xzf oc-mirror.rhel9.tar.gz
```

### 2. configure pull-secret.json

### 3. Create `ImageSetConfiguration`
Manully create an yaml file. Example:
```bash
kind: ImageSetConfiguration
apiVersion: mirror.openshift.io/v2alpha1
mirror:
  platform:
    channels:
    - name: stable-4.17 
      minVersion: 4.17.2
      maxVersion: 4.17.2
    graph: true 
  operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.17 
      packages: 
       - name: aws-load-balancer-operator
       - name: 3scale-operator
       - name: node-observability-operator
  additionalImages: 
   - name: registry.redhat.io/ubi8/ubi:latest
   - name: registry.redhat.io/ubi9/ubi@sha256:20f695d2a91352d4eaa25107535126727b5945bff38ed36a3e59590f495046f0
```

# Performance Requirement

Highly available

# 如何使用 Mirror Registry

# 查看 Mirror Registry
查看 image content source policy （ICSP）
```bash
oc get image.config.openshift.io cluster -o jsonpath='{.spec.registrySources}' | jq
```
# 查看image来源
When using mirrored registries in OpenShift, the system pulls images from your internal (mirrored) registry instead of the original Red Hat registry (`registry.redhat.io`). 

但是，有工具可以查看该 image 真正的源头：
```bash
# 方法一：Check CRI-O Logs (Best Method)
journalctl -u crio --since "10 minutes ago" | grep "Trying to access"
# Example Output: journalctl -u crio --since "10 minutes ago" | grep "Trying to access"

# 方法二：crictl (May Be Misleading)
crictl images
```


!!! note "原理"
    Why Does This Happen?

    - OpenShift uses `imageContentSources` to define mirrors. When an image pull request is made, OpenShift **redirects it to the mirror** - but keeps the original image name in metadata.
    