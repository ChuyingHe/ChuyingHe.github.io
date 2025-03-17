The **operator pattern**  is a way to implement reusable software to manage complex workloads that might have maintenance tasks that can be automated, such as backing up data.

An **operator** typically defines **custom resources (CRs)**. The operator CRs describes the needed information to deploy and manage the workload such as `pod`, `persistentvolume`.


# Operator 类型
!!! info 
    可以将 Operator 比作 一个智能的管家 🕴🏼，它帮助管理 OpenShift 集群中的应用和服务。

    - 就像一个管家会定期检查家里的情况，确保每个房间都保持整洁、设备正常工作，并且根据需要调整家里的布置，Operator 也会监控集群中的资源，确保它们按照预定的配置和规则运行。
    - 如果家里的设备出现故障，管家会立刻修理或者替换。而 Operator 也会在集群中的应用或服务出现问题时，自动修复或重新启动它们。
    - 管家还会根据需要调整家里的布置和规则，Operator 也会根据集群的需求自动进行更新、配置变更或扩展。

## 1. Cluster Operators
**Cluster Operators** 由 **Cluster Version Operator (CVO)** 管理. Cluster Operators 管理整个 Kubernetes 集群级别的资源和功能。它们通常负责关键的基础设施组件，如网络、存储、监控和日志记录等。比如web console， the OAuth server.

**Cluster Operator**可以再Webconsole中找到： **Administration** > **Cluster Settings** > **ClusterOperators**. <br/>--> [此文档](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/operators/cluster-operators-ref#cluster-bare-metal-operator_cluster-operators-ref) 列出了所有built-in ClusterOperators (约35个)。

!!! info "CVO"
    The **Cluster Version Operator (CVO)** installs and updates **Cluster Operators** - as part of the OpenShift installation and update processes.

    CVO 可以用来查看cluster operators的状态：
    ```bash
    oc get clusteroperator
    ```

!!! warning
    |Operator 类型| 管理者 |
    |:-|:-|
    |Cluster Operators|Cluster Version Operator (CVO)|
    |Add-on Operators|Operator Lifecycle Manager (OLM)|

## 2. Add-on Operators
**Add-on Operators** 由核心组件 **Operator Lifecycle Manager (OLM) 管理。** 

Openshift的 webconsole 中可以看到四种级别的 Add-on：

- `Red Hat`: 由 Red Hat 官方提供和支持，达到**企业级**的稳定性和安全标准
- `Certified`: 由第三方开发，并通过 Red Hat 认证，确保了与 OpenShift 的兼容性和质量标准。由第三方提供技术支持
- `Community`: 由开源社区或开发者提供和维护，没有正式的商业支持
- `Marketplace`: 在 [OpenShift Marketplace](https://swc.saas.ibm.com/en-us/redhat-marketplace/documentation/getting-started) 或 [Kubernetes Marketplace](https://marketplace.digitalocean.com/category/kubernetes) 中提供的 Operators。

<img src="../imgs/operatorhub.png" width="700" />

!!! info "其他开源的 Add-on"
    这里[operatorhub.io](https://operatorhub.io/)可以找到可用的 **Add-on Operators**，其中包含各种 级别 的Operator


!!! note
    **Add-on Operators** have a different lifecycle from **Cluster Operators**. The CVO installs and updates **Cluster Operators** in lockstep with the cluster. Administrators use the OLM to install, update, and remove operators independently from cluster updates.

    -> <ins>Cluster 的安装与更新</ins> 与 <ins>(CVO 对) Cluster Operators 的安装与更新</ins> 同步


!!! note "OLM managed Resources"
    - **Catalog source**: Each catalog source resource references an operator repository.
    - **Package manifest**: The OLM creates a **package manifest** for each available operator. 
    - **Operator group**: **Operator groups** define how the OLM presents operators across namespaces.
    - **Subscription**: Cluster administrators create **subscriptions** to install operators.
    - **Operator**: The OLM creates **operator** to store information about installed operators.
    - **Install plan**: The OLM creates **install plan** as part of the installation and update process. When requiring approvals, administrators must approve **install plans**.
    - **Cluster service version (CSV)**: Each version of an operator has a corresponding CSV. The CSV contains the information that the OLM requires to install the operator.




### Install Operator in CLI
!!! note "Steps to install an Operator"
    1. Locate the operator to install.
    2. Review the operator and its documentation for installation options and requirements.
        - Decide the **update channel** to use.
        - Decide the **installation mode**. For most operators, you should make them available to all namespaces.
        - Decide to deploy the operator workload to an existing **namespace** or to a new namespace.
        - Decide whether the Operator Lifecycle Manager (OLM) applies updates automatically, or requires an administrator to approve updates.
    3. Create an **operator group** if needed for the installation mode.
    4. Create a namespace for the operator workload if needed.
    5. Create the operator **subscription**.
    6. Review and test the operator installation.


#### 1. Examine the operator
```bash
# Examine catalog sources 
oc get catalogsource -n openshift-marketplace

# List the package manifests to know which operators are available for installation. 
# a.k.a List Operators in current Catalog
oc get packagemanifests

# check detail of the operator 'lvms-operator'
oc describe packagemanifest lvms-operator -n openshift-marketplace
```

!!! note "example"
    <img src="../imgs/operator_catalogs.png" width="700" />
    <img src="../imgs/operators.png" width="700" />

!!! note "describe Operator details"
    <img src="../imgs/operator_detail_1.png" width="700" />
    <img src="../imgs/operator_detail_2.png" width="700" /> 

    - (1), (2): The **catalog source** and **namespace** for the operator,
    - (3), (7): **available channels** and **CSVs** to decide which upgrade path to use.
    - (4), (6): description and links for installation
    - (5) install modes


#### 2. [OPTIONAL] Create Operator Group 

- Many operators recommend to use the existing `openshift-operators` namespace, or require specific namespaces.
- The `openshift-operators` namespace contains a `global-operators` **operator group**. Operators installed in this namespace, uses this group

Optionally, you can also - Create an operator group:
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: name
  namespace: namespace  # where OperatorGroup itself is located. Operator will be here too
spec:
  targetNamespaces:     # where the operator should be available or operate
  - namespace
```

#### 3. Create a subscription
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: lvms-operator
  namespace: openshift-storage
spec:
  channel: stable-4.14 
  name: lvms-operator 
  source: do280-catalog-cs 
  installPlanApproval: Automatic  # ‼️ or Manual
  sourceNamespace: openshift-marketplace
```

#### 4. Approve Install Plan
```bash
# check status
oc describe operator file-integrity-operator

# If the install plan mode is set to 'Manual' in the Subscription, 
# then you must manually approve the install plan:
oc patch installplan install-pmh78 --type merge -p \
    '{"spec":{"approved":true}}' -n openshift-file-integrity
```

!!! note "oc patch"
    `--type` has 4 possible values:

    - `merge`: DEFAULT, only change the given value
    - `strategic-merge`: similar to `merge`, but specifically designed for Kubernetes resources.
    - `replace`: replace the entire resource
    - `json`: allows you to apply changes based on the JSON Patch standard

⚠️ With an 'Automatic' install plan mode, the OLM applies updates as soon as they are available.

!!! warning "Troubleshooting"
    Some operators require additional steps to install or update. Review the documentation to validate whether you performed all necessary steps, and to learn about support options.


## 3. Other Operators
Software providers can create software that follows the operator pattern, and then distribute the software as manifests, Helm charts, or any other software distribution mechanism.

# View installed Operators
The **Installed Operators** page lists the installed **cluster service version (CSV)** as this:

<img src="../imgs/csv.png" width="700" />

or

```bash
oc get csv
```

# Using installed Operators
**Custom resources** are the most common way to interact with operators. You can create custom resources:

<img src="../imgs/operator_installed.png" width="700" />

You can create by **view**:

<img src="../imgs/operator_new_view.png" width="500" />

or by **yaml**:

<img src="../imgs/operator_new_yaml.png" width="500" />

# Debugging
for troubleshooting you can check here:

<img src="../imgs/operator_deployments.png" width="600" />

Check the following:

- The status of Kubernetes workload resources, such as deployments or stateful sets 
- Pod logs & status
- Events