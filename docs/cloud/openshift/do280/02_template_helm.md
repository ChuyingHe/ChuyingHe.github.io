# 1. OpenShift Template
Template（模板） 是一种资源定义，允许你通过参数化的方式来创建一组 Kubernetes 或 OpenShift 资源。这种机制可以简化和复用应用程序的部署过程。

|Command|Description|
|:-|:-|
|`oc get templates -n openshift`|list templates in `openshift` namespace|
|`oc get template cache-service -o yaml -n openshift`|打印模版到 `yaml` 文件|
|`oc describe template cache-service -n openshift`|查看模版的核心要素|
|`oc process --parameters cache-service -n openshift`|只看模版的 **Parameters**|
|`oc process --parameters -f my-template.yaml`|只看模版文件的 **Parameters**|

## 模板的核心要素
一个 OpenShift Template 通常包含以下几个部分：

- **Metadata**: 定义模板的元数据，比如名字和标签。
- **Parameters**: 参数是模板的关键，允许用户在实例化模板时传递值。支持默认值，可以为某些参数提供动态替换功能。--> the values that can be customized
- **Objects**: 包含模板将生成的具体资源，比如 Deployment、Service、ConfigMap 等。
- **Labels**: 为所有生成的资源统一添加标签，方便管理和选择。



## Assign Parameters to Template
1. value with `-p`
    ```bash
    oc process my-cache-service \
    -p TOTAL_CONTAINER_MEM=1024 \
    -p APPLICATION_USER='cache-user' \
    -o yaml \
    > my-cache-service-manifest.yaml
    ```
2. values within file:
    ```bash
    oc process my-cache-service \
    --param-file=my-cache-service-params.env \
    -o yaml \
    > my-cache-service-manifest.yaml
    ```
## Create src with Template

```bash
oc process my-cache-service \
  --param-file=my-cache-service-params.env | oc apply -f -
```

## Update src with Template
To compare the results of applying a different parameters file to a template against the live resources:
```bash
oc process my-cache-service -o yaml \
  --param-file=my-cache-service-params-2.env | oc diff -f -
```

## CLIs
|Command|Description|
|:-|:-|
|`oc new-app --template=cache-service -p APPLICATION_USER=my-user`|create new src from the template|
|`oc process my-cache-service -p APPLICATION_USER=user1 -o yaml > my-cache-service-manifest.yaml`|generate **manifest** YAML from the **template** with the given parameter values. |
|`oc process -f my-cache-service.yaml -p APPLICATION_USER=user1 -o yaml > my-cache-service-manifest.yaml`|`-f`: generate **manifest** YAML from the **template file** with the given parameter values. |


# 2. Helm
[check this](../../helm/helm-1.md)


!!! note "Helm vs Kustomize"
    Kustomize 和 Helm Chart 是两种用于管理 Kubernetes 配置的工具，它们在功能和使用方式上有所不同，但都旨在简化 Kubernetes 应用的部署和管理。

    - Kustomize 是一个原生支持 Kubernetes 的配置管理工具，允许用户不使用模板而直接调整和复用 YAML 文件。
    - Helm Chart 包括 Kubernetes 的资源模板（如 Deployment、Service 等）和一个 values 文件，通过这个文件可以对模板中的变量进行配置。