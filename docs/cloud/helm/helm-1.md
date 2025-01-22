
Helm is a command-line application. It introduces the concept of **charts**. A **chart** is a package that describes a set of **Kubernetes resources** that you can deploy.

The following diagram shows the structure of a minimal Helm chart:

```bash
sample/
├── Chart.yaml
├── templates
|   |── example.yaml
└── values.yaml
```

- The `Chart.yaml` contains chart metadata, such as the name and version of the chart.
- The `/templates` directory contains files that define application resources such as deployments.
- The `values.yaml` file contains default values for the chart.



# create & check

|命令||
|:-|:-|
|`helm create myChart` |to create a new chart |
|`helm pull xxx` |download a chart from a repository |
|`helm dependency update` |to add dependencies and lock the versions |
|`helm show chart myChart` |show info of a chart |
|`helm show values myChart` |show default values of a chart |
| |use `--version` to choose a specific version |


# Deployment
take release name = `myChartApp`.

|命令||
|:-|:-|
|`helm install myChartApp .` |deploy the app from <b>current directory</b> |
| |`helm install example-app do280-repo/etherpad -f values.yaml --version 0.0.6` |
|`helm uninstall myChartApp` |delete deployment/release |
|`helm list` |list all deployment |
|`helm status myChartApp` |check status of named release |
|`helm history myChartApp` |check release history |
|`helm rollback myChartApp` |roll back to the previous release |
|`helm upgrade myChartApp ...` |`helm upgrade myChartApp do280-repo/etherpad --version 0.0.7`  upgrade version to 0.0.7 <br/>`helm upgrade myChartApp do280-repo/etherpad -f values2.yaml`  upgrade values <br/>|
|`helm template myChartApp helm-directory > base/deployment.yaml` |extract the object definition from <b>Helm Chart</b> into Kustomize's `/base/deployment.yaml`|

# Helm Repository
The distributed community Helm chart repository is located at [Artifact Hub](https://artifacthub.io/packages/search?kind=0) and welcomes participation.


The following commands change only local configuration:

|命令||
|:-|:-|
|`helm repo add openshift-helm-charts https://charts.openshift.io/` |add a new Helm chart repository(named `openshift-helm-charts`) to your local Helm configuration. |
|`helm repo list` |List Helm chart repositories.|
|`helm repo update ` |Update Helm chart repository.|
|`helm search repo` |search for available charts in the `openshift-helm-charts` repository|
|`helm search repo openshift-helm-charts` |search for available charts in the all the repositories<br/>- Add `--versions` flag to list all versions even though its the same chart|
|`helm repo remove REPOSITORY1_NAME REPOSITORY2_NAME …​	` |Remove repositories |

!!! note "Example"
    result of `helm search repo openshift-helm-charts`:

    <img src='../helm_search.png' width="800" />