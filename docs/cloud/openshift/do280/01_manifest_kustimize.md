**imperative commands 命令式** means using CLIs directly. Disadvantage:

- Impaired reproducibility
- Lacking version control
- Lacking support for GitOps

**declarative commands 声明式** is a way to manage resources using manifest file (`YAML` or `JSON`)


Both Kubernetes and Openshift have both **imperative** and **declarative** options. However in production, we recommend to use **imperative** for developing and experimenting, **declarative** for deploying, maintaining and documenting. 

# 1. Manifest / 配置
## Create Manifest

Where to get the YAML file?

1. Use the YAML view of a resource in the web console.
2. Use imperative commands with the `--dry-run=client` option to generate manifests. E.g.:
```sh
kubectl create deployment hello-openshift -o yaml \
    --image registry.ocp4.example.com:8443/redhattraining/hello-world-nginx:v1.0 \
    --save-config \ 
    --dry-run=client \ 
    > ~/my-app/example-deployment.yaml
```

    - The `--save-config` saves the current config (in JSON format) in the annotation `kubectl.kubernetes.io/last-applied-configuration`. 其实就是一个版本控制的作用，用来记录上一次deployment的配置，以确保资源状态与配置文件一致。
    - The `--dry-run=client` option prevents the command from creating resources in the cluster.

!!! tip "Doc"
    To see the details of the field, for example `deployment.spec.template.spec`:
    ```bash
    kubectl explain deployment.spec.template.spec
    ```

!!! tip "Multiple src in single YAML Manifest"
    Use a line of `---` to separate the resources


## Customize Manifest
你可以修改以下这些内容：

1. Remove empty fields
2. Changing attributes such as：
    - `namespace`
    - `spec.replicas` 
    - `spec.template.spec.containers[].ports.containerPort`


## Create src with Manifest
- Creating with 1 Manifest file (or a URL):
  ```bash
  kubectl create -f resource.yaml
  kubectl apply -f resource.yaml

  kubectl create -f https://example.com/example-apps/deployment.yaml
  ```
- Create with all Manifest files under one directory
  ```bash
  kubectl create -R -f ~/my-app
  ```

## Update src with Manifest
更新资源：

```bash
kubectl apply -f resource.yaml
```

!!! note "Compare: `apply` vs `create`"
    ➡️ `oc apply` is more powerful

    ||`kubectl apply`|`kubectl create`|
    |:--|:--|:--|
    |Type|**declarative**|**imperative**|
    |consider the<br/>current state<br/> of a **Live Resource**?|Yes, compare:<br/> 1.Live configuration <br/>2.Manifest file <br/>3.Configuration in the annotation `last-applied-configuration`|No|
    |can it update<br/>**Live Resource**?|Yes|No|
    |⚠️ auto annotated in <br/> `kubectl.kubernetes.io/last-applied-configuration`|yes|no, you need to add `--save-config` flag|


!!! warning "rollout"
    If an updated Manifest changes only values in **Secret** or **ConfigMap**, the updated Manifest does NOT generate new pods automatically. Instead:

    - use `oc rollout restart deployment/deployment-name` to <ins>force the **restart**</ins> → 是重启，不是回滚到上一个版本
    - if `spec.replica` == 1, you can also delete the previous pod.


## Patch Manifest
The `oc patch` command updates or adds fields in an existing object:

- from json
  ```bash
  oc patch deployment hello -p \
    '{"spec":{"template":{"spec":{"resources":{"requests":{"cpu": "100m"}}}}}}'
  ```
- from a patch file:
  ```bash
  oc patch deployment hello --patch-file \
    ~/volume-mount.yaml
  ```

    !!! note "patch 文件"
        上述例子中，oc patch 使用的 patch 文件不需要是一个完整的 Deployment 定义。它只需要包含你想要修改的部分。Patch 文件可以是 JSON 或 YAML 格式。举例`volume-mount.yaml`
        ```yaml
        spec:
          template:
            spec:
              containers:
                - name: hello-container # 指定要修改的容器
                  volumeMounts:
                    - name: my-volume
                      mountPath: /data
              volumes:
                - name: my-volume
                  emptyDir: {}
        ```

## Validate Manifest

- `--dry-run=server`: submits a server-side request without persisting the resource.
- `--validate=true`: uses a schema to validate the input and fails the request if it is invalid. 比如json/yaml语法，数据类型，和必须要有的key-value

!!! warning "Compare: `server` vs `client`"
    **两者都不会真正创建或更新资源。**

    ||`--dry-run=client`|`--dry-run=server`|
    |:--|:--|:--|
    |where?|在本地进行验证，不会与 Kubernetes 服务器交互。|在服务器端进行验证。它将把配置文件发送到 Kubernetes API 服务器，由服务器检查文件内容是否在集群中有效（例如，字段值是否正确、资源是否存在等）|
    |suitable for|检查 YAML 文件的基本格式和结构，适合快速检查配置文件是否有明显错误|更严格，适合需要精确验证的场景|

## Compare Manifest
To review differences between live objects and manifests in current directory(`.`). 
<br/>→ 只是比较，不进行实质更改

```bash
oc diff -f .
# or
kubectl diff -f .
```

## Delete Resource with Manifest
```bash
oc delete -f .
```

OpenShift looks at the **resource types** and **names** in the YAML files.

!!! danger
    Live Resources and the YAML file does NOT have to be identical!

# 2. Kustomize
<img src="../imgs/kustomize_logo.jpg" width="300" />

[Walk through video](https://www.youtube.com/watch?v=spCdNeNCuFU&ab_channel=DevOpsJourney)

**Kustomize** 是一个开源的Kubernetes **配置管理工具**，用于对Kubernetes 清单文件进行自定义和修改。 它允许用户通过分层和声明式的方式管理和定制应用程序的配置，而无需直接修改原始的清单文件，促进了配置的复用和可维护性。我们可以用Kustomize配置多个环境，比如：

- development
- staging
- testing
- production


!!! info
    Both `kubectl` and `oc` commands integrated the **Kustomize** tool.


## Files
Kustomize根目录下有两个文件夹：一个`base/`，一个`overlays/`
### `Base`
The `base/kustomization.yaml` file includes a list that includes all resource files under `base/`. E.g.:

<table>
  <tr>
    <th>  </th>
    <th>Example 1:</th>
    <th>Example 2:</th>
  </tr>
  <tr>
    <td>base/<br/>kustomization.yaml</td>

    <td>
    ```yaml
    # kustomization.yaml
    apiVersion: kustomize.config.k8s.io/v1beta1
    kind: Kustomization
    resources:
    - configmap.yaml
    - deployment.yaml
    - secret.yaml
    - service.yaml
    - route.yaml
    ```
    </td>
    <td>

    ```yaml
    # kustomization.yaml
    apiVersion: kustomize.config.k8s.io/v1beta1
    kind: Kustomization
    resources:
    - database
    - exoplanets # ⬅️ 这是个文件夹！
    ```

    </td>
  </tr>
  <tr>
    <td>File structure:</td>
    <td>
    ```bash
    .
    ├── base
    │   ├── configmap.yaml
    │   ├── deployment.yaml
    │   ├── secret.yaml
    │   ├── service.yaml
    │   └── kustomization.yaml
    └── overlays  
        ├── kustomization.yaml
        └── patch-replicas.yaml
    ```
    </td>
    <td>
    ```bash
    .
    ├── base
    │   ├── database 
    │   │   ├── configmap.yaml
    │   │   ├── deployment.yaml
    │   │   ├── kustomization.yaml
    │   │   ├── secret.yaml
    │   │   └── service.yaml
    │   ├── exoplanets 
    │   │   ├── configmap.yaml
    │   │   ├── deployment.yaml
    │   │   ├── kustomization.yaml
    │   │   ├── route.yaml
    │   │   ├── secret.yaml
    │   │   └── service.yaml
    │   └── kustomization.yaml 
    └── overlays  
        └── production
            ├── kustomization.yaml
            └── patch-replicas.yaml
    ```
    </td>
  </tr>
  
</table>





### `Overlays`
**Overlays** overwrites some setting without modifying the original **Base**. Each **Overlays** contains a `kustomization.yaml` file.

<img src="../imgs/kustomize-directory-structure.svg" />


!!! note "Relationship: Base + Overlays"
    - The `kustomization.yaml` file can refer to **one or more** directories as bases. 
    - Multiple overlays can use a common base kustomization directory.


```txt
# result of `tree .`
base
├── configmap.yaml
├── deployment.yaml
├── secret.yaml
├── service.yaml
├── route.yaml
├── kustomization.yaml
overlay
└── development
    └── kustomization.yaml
└── testing
    └── kustomization.yaml
└── production
    ├── kustomization.yaml
    └── patch.yaml
```


```yaml
# overlays/development/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev-env
resources:
- ../../base
```


!!! note "extra Attributes"
    These are the extra **Attributes** provided by Kustomize to use in `kustomization.yaml`:

    |**Field**|**Description**|
    |:--|:--|
    |`namespace`|Set a specific namespace for all resources.|
    |`namePrefix`|Add a prefix to the name of all resources.|
    |`nameSuffix`|Add a suffix to the name of all resources.|
    |`commonLabels`|Add labels to all resources and selectors.|
    |`commonAnnotations`|Add annotations to all resources and selectors.|


### Patch
patch 是 `overlays` 用于修改 `base` 资源的具体方式，比如 `overlays/production/patch.yaml` 文件帮助实现了 `production` overlay。

The patch mechanism has several important keys: `patch`, `target` and `path`.

**Way 1: `patch` and `target`**

```yaml
# overlays/production/patch.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: test-env

# list of patches
patches: 
- patch: |-
    - op: replace             # operation
      path: /metadata/name
      value: frontend-test
  target:                     # target
    kind: Deployment
    name: frontend
- patch: |-                   
    - op: replace
      path: /spec/replicas
      value: 15
  target:
    kind: Deployment
    name: frontend
resources:                    # used "base"
- ../../base
commonLabels:                 # add label to all the resources
  env: test
```

**Way 2: `path`**
```yaml
# patch.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-prod
spec:
  replicas: 5
```


```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: prod-env

# list of patches
patches:
- path: patch.yaml          # the name of Patch File
  target:                   # target
    kind: Deployment
    name: frontend
  options:
    allowNameChange: true   # this OPTION enables kustomization to update the name by using a patch YAML file
resources:                  # used "base"
- ../../base
commonLabels:               # add label to all the resources
  env: prod
```

 
## CLIs

Example: I want to use `overlay/production` now:

|<div style="width: 300px">Command</div>|Description|
|:--|:--|
|`oc/kubectl kustomize overlay/production`|打印查看该 Overlay 会生成的manifests |
|`oc/kubectl apply -k overlay/production`|use `overlay/production` layer. <br/>- Update/create resources.<br/>- `-k` flag means: applies a **kustomization**<br/>- 先使用 Kustomize 生成最终的 YAML 配置（但不输出），然后将其应用到 Kubernetes 集群|
|`oc/kubectl apply -k base`|use `base` layer|
|`oc/kubectl delete -k overlay/production`|delete resources that were deployed by using **Kustomize**|


## Generator
Kustomize has **configMapGenerator** and **secretGenerator** fields that generate `configmap` and `secret` resources. **Generators** help to manage the content of `configmaps` and `secrets`, by taking care of encoding and including content from other sources.

### configMapGenerator

Definition: The Kustomize's `configmapGenerator` can generates `cm` in 3 ways. see example：
```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: hello-stage
resources:
- ../../base
configMapGenerator:
- name: configmap-1   # 1️⃣ from file *.properties 
  files:
    - application.properties
- name: configmap-2   # 2️⃣ from env *.env
  envs:
    - configmap-2.env
- name: configmap-3   # 3️⃣ from literal key-value pair
  literals:
    - msg="Welcome!"
    - enable="true"
```

Usages: In the deployment file uses the generated `configmap`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  labels:
    app: hello
    name: hello
spec:
...output omitted...
    spec:
      containers:
      - name: hello
        image: quay.io/hello-app:v1.0
        env:
        - name: MY_MESSAGE
          valueFrom:
            configMapKeyRef:
              name: configmap-3   # <-- Use the CM by name
              key: msg
        - name: MSG_ENABLE
          valueFrom:
            configMapKeyRef:
              name: configmap-3
              key: enable
```

!!! note "Note"
    - By changing the `configMapGenerator`, Kustomize will generate new `configmap` in the next `oc apply -k` process
    - the `configmap` generated by **Kustomize Generator** behaves a bit differently → Kustomize appends a <ins>hash</ins> to the name like this:
    ```yaml
    apiVersion: v1
    data:
      application.properties: |
        Day=Monday
        Enable=True
    kind: ConfigMap
    metadata:
      name: configmap-1-5g2mh569b5    # 1️⃣ from file
    ---
    apiVersion: v1
    data:
      Enable: "True"
      Greet: Welcome
    kind: ConfigMap
    metadata:
      name: configmap-2-92m84tg9kt    # 2️⃣ from env
    ---
    apiVersion: v1
    data:
      description: literal key-value pair
      name: configmap-3
    kind: ConfigMap
    metadata:
      name: configmap-3-k7g7d5bffd    # 3️⃣ from literal key-value pair
    ---
    ...output omitted...
    ```

!!! danger
    假设我们修改了`configMapGenerator`中的一个值，某个`Deployment`在使用生成的`config`。然后我用`oc apply -k overlays/production`应用这个修改，那么我将得到：

    - 一个拥有新哈希值的cm：`hello-3-696dm8h728`
    - 一个使用新cm的新Deployment：`hello-55bc55ff9-hrszh`

### secretGenerator
The Kustomize's `secretGenerator` can generates `secret` also in 3 ways: file, env, or literals.

### generatorOptions
This `generatorOptions` can define alternative behavior of the `*Generators`. For example:

#### 1. disable <ins>hash suffix</ins>

**Why hash suffix are needed?**<br>
我们使用`oc/kubectl apply -k overlays/production`应用修改时，会自动生成带有新哈希suffix的resources，保证了修改被使用到。

But in some case, the <ins>hash suffix</ins> is not needed. I can disable it with the `generatorOptions`:

```yaml
generatorOptions:     # <-- The Option
  disableNameSuffixHash: true
configMapGenerator:   # The Generators
- name: my-configmap
  literals:
    - name="configmap-3"
    - description="literal key-value pair"
```
!!! warning
    优先级：`Kustomize` 的规则是 **局部设置 > 全局设置**，所以局部的 `disableNameSuffixHash: false` 会生效。

    ```yaml
    apiVersion: kustomize.config.k8s.io/v1beta1
    kind: Kustomization
    generatorOptions:   
      disableNameSuffixHash: true       # Global
    configMapGenerator:
      - name: global-config
        literals:
          - key1=value1
        options:
          disableNameSuffixHash: false  # Local
          annotations:
            createdBy: "local-options"
          labels:
            environment: "test"
    ```


#### 2. Add `labels`/`annotations`

```yaml
# 这个标签将添加到当前文件中所有的 ConfigMap 和 Secret 中。
generatorOptions:
  labels:
    fruit: apple
  annotations:
    note: project-on-cloud
```

# Tips & Tricks
!!! tip "watch"
    `watch -d oc get deployments,pods` 
    
    The `watch` tool comes from the `procps-ng` package, 是一个 Linux/Unix 系统自带的命令行工具，它用于周期性地（默认 2 秒）执行一个命令并实时刷新输出。

    - `-d` highlights the difference
    - Use `Ctrl+C` to exit the watch model


# CLIs
Print the contents of the secret:

```bash
oc extract secret/db-secrets-55cbgc8c6m --to=-
```

- The `--to` option specifies the destination for the extracted files.
- The `-` (dash) is a common convention in Unix-like systems that represents standard output (stdout).

# Q&A

## 1.Deployment & updated Secret
Q: In Kubernetes, if I update the **secret** value, will the **deployment** get the updates automatically?

A: No. **Secrets** are mounted as files in **pods** (if used as volumes) or injected as environment variables.

Q: whats the difference between these 2 methods? Does it make difference when Secret is a file or just Env?
    1. oc delete pod/xxx
    2. oc rollout restart deployment/xx

A:?