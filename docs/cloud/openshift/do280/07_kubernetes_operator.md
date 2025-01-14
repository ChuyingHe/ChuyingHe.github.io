# 1. Kubernetes Operator
The **operator pattern**  is a way to implement reusable software to manage complex workloads that might have maintenance tasks that can be automated, such as backing up data.

An **operator** typically defines **custom resources (CRs)**. The operator CRs describes the needed information to deploy and manage the workload such as `pod`, `persistentvolume`.


## Operator 类型
### 1. Cluster Operators
**Cluster Operators** 由 **Cluster Version Operator (CVO)** 管理. Cluster Operators 管理整个 Kubernetes 集群级别的资源和功能。它们通常负责关键的基础设施组件，如网络、存储、监控和日志记录等。比如web console， the OAuth server.

**Cluster Operator**可以再Webconsole中找到： **Administration** > **Cluster Settings** > **ClusterOperators**.

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

### 2. Add-on Operators
**Add-on Operators** 由核心组件 **Operator Lifecycle Manager (OLM) 管理。** 

Openshift的 webconsole 中可以看到四种级别的 Add-on：

- Red Hat
- Certified
- Community
- Marketplace

<img src="../imgs/operatorhub.png" width="700" />

!!! info "其他开源的 Add-on"
    这里[operatorhub.io](https://operatorhub.io/)可以找到可用的 **Add-on Operators**


!!! note
    **Add-on Operators** have a different lifecycle from **Cluster Operators**. The CVO installs and updates **Cluster Operators** in lockstep with the cluster. Administrators use the OLM to install, update, and remove operators independently from cluster updates.


!!! note "OLM managed Resources"
    - **Catalog source**: Each catalog source resource references an operator repository.
    - **Package manifest**: The OLM creates a **package manifest** for each available operator. 
    - **Operator group**: **Operator groups** define how the OLM presents operators across namespaces.
    - **Subscription**: Cluster administrators create **subscriptions** to install operators.
    - **Operator**: The OLM creates **operator** to store information about installed operators.
    - **Install plan**: The OLM creates **install plan** as part of the installation and update process. When requiring approvals, administrators must approve **install plans**.

#### Install Operator in CLI
**1. Examine the operator**<br/>
```bash
# Examine catalog sources 
oc get catalogsource -n openshift-marketplace

# List the package manifests to know which operators are available for installation. 
oc get packagemanifests

# check detail of the operator 'lvms-operator'
oc describe packagemanifest lvms-operator -n openshift-marketplace
```

**2. Create Operator Group [Optional]**<br/>
- Many operators recommend to use the existing `openshift-operators` namespace, or require specific namespaces.
- The `openshift-operators` namespace contains a global-operators **operator group**. Operators installed in this namespace, uses this group

Optionally, you can also create an operator group:
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: name
  # Operators follow the operator group in the namespace that they are deployed in
  namespace: namespace
spec:
  # List the namespaces that the operator monitors 
  targetNamespaces:
  - namespace
```

**3. Create a subscription**<br/>
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
  installPlanApproval: Automatic
  sourceNamespace: openshift-marketplace
```

**4. Approve Install Plan**<br/>
```bash
# check status
oc describe operator file-integrity-operator

# approve install plan
oc patch installplan install-pmh78 --type merge -p \
    '{"spec":{"approved":true}}' -n openshift-file-integrity
```

!!! warning "Troubleshooting"
    Some operators require additional steps to install or update. Review the documentation to validate whether you performed all necessary steps, and to learn about support options.


### 3. Other Operators
Software providers can create software that follows the operator pattern, and then distribute the software as manifests, Helm charts, or any other software distribution mechanism.

## View installed Operators
The **Installed Operators** page lists the installed **cluster service version (CSV)** as this:

<img src="../imgs/csv.png" width="700" />

## Using installed Operators
**Custom resources** are the most common way to interact with operators. You can create custom resources:

<img src="../imgs/operator_installed.png" width="700" />

You can create by **view**:

<img src="../imgs/operator_new_view.png" width="500" />

or by **yaml**:

<img src="../imgs/operator_new_yaml.png" width="500" />

## Debugging
for troubleshooting you can check here:

<img src="../imgs/operator_deployments.png" width="600" />

Check the following:

- The status of Kubernetes workload resources, such as deployments or stateful sets 
- Pod logs & status
- Events