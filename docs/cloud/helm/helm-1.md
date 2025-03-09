
👍 [Helm 命令文档](https://helm.sh/docs/helm/)


Helm is a command-line application. It introduces the concept of **charts**. A **chart** is a package that describes a set of **Kubernetes resources** that you can deploy.

The following diagram shows the structure of a minimal Helm chart:

```bash
sample/
├── Chart.yaml
├── templates
|   |── example.yaml
└── values.yaml
```

- The `Chart.yaml` contains chart `metadata`, such as **name** and **version**
- The `templates/` directory contains files that define **application resources** such as `deployments`
- The `values.yaml` file contains default `values` for the chart



# create & check

|命令||
|:-|:-|
|`helm create myChart` |to create a new chart |
|`helm pull xxx` |download a chart from a **repository**（即charts的数据库） |
|`helm show chart myChart` |show info of a chart |
|`helm show values myChart` |show default values of a chart |
| |use `--version` to choose a specific version |


## 依赖管理
|命令||
|:-|:-|
| `helm dependency update`| 根据 Chart.yaml 中的依赖关系，下载或更新依赖的 Chart 到 charts/ 目录。|
| `helm dependency build`| 使用 Chart.lock 文件中的版本信息，重新下载依赖的 Chart。|
| `helm dependency list`| 列出当前 Chart 的所有依赖项及其状态。|
| `helm dependency prune`| 删除不再需要的依赖 Chart。|

# Deployment

- `[RELEASE]`: release/app
- `[CHART]`: chart name

|命令||
|:-|:-|
|`helm install [RELEASE] .` |deploy the app from <b>current directory</b> |
| |`helm install myChartApp do280-repo/etherpad -f values.yaml --version 0.0.6` |
|`helm uninstall [RELEASE]` |delete deployment/release |
|`helm list` |list all deployment |
|`helm status [RELEASE]` |check status of named release |
|`helm history [RELEASE]` |check release history |
|`helm rollback [RELEASE]` |roll back to the previous release |
|`helm upgrade [RELEASE] [CHART] [flags]` |`helm upgrade myChartApp do280-repo/etherpad --version 0.0.7`  upgrade version to 0.0.7 <br/>`helm upgrade myChartApp do280-repo/etherpad -f values2.yaml`  upgrade values <br/>|
||Flags 比如 `-f values.yaml`|
|`helm template [RELEASE] helm-directory > base/deployment.yaml` |extract the object definition from <b>Helm Chart</b> into Kustomize's `/base/deployment.yaml`|

!!! info
    to use `values.yaml` you can use either `--values` or `-f` 

# Helm Repository
The distributed community Helm chart repository is located at [Artifact Hub](https://artifacthub.io/packages/search?kind=0) and welcomes participation.


The following commands change only local configuration:

|命令||
|:-|:-|
|`helm repo add openshift-helm-charts https://charts.openshift.io/` |add a new Helm chart repository(named `openshift-helm-charts`) to your local Helm configuration. |
|`helm repo list` |List Helm chart repositories.|
|`helm repo update ` |Update Helm chart repository.|
|`helm search repo` |List ALL available charts in the ALL repos<br/>- Add `--versions` flag to list all versions even though its the same chart|
|`helm search repo openshift-helm-charts` |search for available charts in the `openshift-helm-charts` repository|
|`helm repo remove REPOSITORY1_NAME REPOSITORY2_NAME …​	` |Remove repositories |

!!! note "Example"
    result of `helm search repo openshift-helm-charts`:

    <img src='../helm_search.png' width="800" />

!!! note
    某个chart中有这些values：
    ```bash
    [student@workstation ~]$ helm show values do280-repo/etherpad --version 0.0.6

    replicaCount: 1
    defaultTitle: "Labs Etherpad"
    defaultText: "Assign yourself a user and share your ideas!"

    image:
        repository: etherpad
        name:
        tag:
    ```

    自定义值 `values.yaml`：
    ```
    image:
        repository: registry.ocp4.example.com:8443/etherpad
        name: etherpad
        tag: 1.8.18
    ```