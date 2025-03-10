# 1. ResourceQuota

!!! info "Workload Resource Limit"
    For resource limitating purpose, workloads can specify the following properties:
    - **Resource limits**: to limit the resources that a workload consumes.
    - **Resource requests**: to declare the minimum required resources, to prevent deployments of new workloads if the cluster has insufficient resources.

Cluster adminstrator might needs more control, by dividing workloads into **namespaces**. `ResourceQuota` defines constraint that applies to the whole **namespace**, more basic knowledge check [here](/cloud/k8s/ckad/ckad-1/#resourcequota)。

# 2. ClusterResourceQuota
**Cluster administrators** can use `ClusterResourceQuota` to apply restrictions across namespaces

## 创建

### 1. YAML
```yaml
apiVersion: quota.openshift.io/v1
kind: ClusterResourceQuota
metadata:
  name: example
spec:
  quota:
    hard:
      limits.cpu: 4
  selector:
    annotations: {}
    labels:
      matchLabels:
        kubernetes.io/metadata.name: example
```
### 2. Web Console
Administration > CustomResourceDefinitions


### 3. CLI
```bash
oc create clusterresourcequota example --project-label-selector=group=dev --hard=requests.cpu=10
```

## 查看
Users might not have read access to `clusterresourcequota`. OpenShift creates resources called `AppliedClusterResourceQuota` in namespaces that are affected by cluster resource quotas. 

```bash
oc describe AppliedClusterResourceQuota -n example-2
```


# 3. LimitRange
`LimitRange` defines constraint that applies to Workload, for example: `containers`, `pods`, `images`, `imagestream`, and `pvc`. 

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: default
spec:
  limits:
    - default:
        memory: 512Mi
      defaultRequest:
        memory: 256Mi
      type: Container
```

Limit ranges can specify the following limit types:

- `default`: default limits for workloads,  eliminate a need to declare limits explicitly in each workload. 
- `defaultRequest`: default requests for workloads<br/> ⚠️ `default` >= `defaultRequest`
- `max`: the maximum value of both requests and limits. Users cannot create workloads that declare limits or that make resource requests over the maximum.<br>⚠️ Consider allowing users who create workloads to edit maximum limit ranges.
- `min`: the minimum value of both requests and limits.<br/>To ensure that users create workloads with enough requests and limits. 
- `maxLimitRequestRatio`: the relationship between limits and requests. If you set a ratio of two, then the resource limit cannot be more than twice the request.


!!! warning
    When a `ResourceQuota`/`ClusterResourceQuota` is present, all workloads must specify the corresponding limits and requests. When you set the `default` and `defaultRequest` keys in a `LimitRange`, workloads use the requests and limits from the limit range by `default`. So you will be able to create workloads without seeing this error:

    ```bash
    ...output omitted...
    13s         Warning   FailedCreate        replicaset/example-74c57c8dff   Error creating: pods "example-74c57c8dff-rzl7w" is forbidden: failed quota: example: must specify limits.cpu for: hello-world-nginx; limits.memory for: hello-world-nginx; requests.cpu for: hello-world-nginx; requests.memory for: hello-world-nginx
    ...output omitted...
    ```

!!! info 
    `LimitRange` do not affect existing pods. 

|CLI||
|:-|:-|
|`oc get event --sort-by .metadata.creationTimestamp`||
|`oc set resources deployment example --limits=cpu=[new-cpu-limit]`||



# 4. Project Template

!!! warning "OC Project vs k8s Namespace"
    OpenShift introduces `projects` to improve security and users' experience of working with `namespaces`. The OpenShift API server adds the `Project` resource type. 
    
    When you make a query to list `projects`, the API server lists `namespaces`, filters the visible `namespaces` to your user, and returns the visible `namespaces` in `project` format.


OpenShift introduces the `ProjectRequest` resource type. When you create a project request, the OpenShift API server creates a `namespace` from a **Project Template**. By using a **Project Template**, cluster administrators can customize `namespace` creation. For example, cluster administrators can ensure that new `namespaces` have specific permissions, `ResourceQuota`, or `LimitRange`.

<img src="../imgs/project_ns.png" width="200" />

## Resources
You can add any namespaced resource to the **Project Template**. For example:

- **Role & RoleBinding** <br/>
  Add roles and role bindings to the template to grant specific permissions in new projects. The default template grants the admin role to the user who requests the project.
- **ResourceQuota & LimitRange** <br/>
  Add resource quotas to the project template to ensure that all new projects have resource limits. Therefore, better create also `LimitRange` when you have `ResourceQuota` - to reduce the effort for workload creation.
- **NetworkPolicy**  <br/>
  Add network policies to the template to enforce organizational network isolation requirements.

### Creation
Create a file with an initial template:
```bash
oc adm create-bootstrap-project-template -o yaml > file
```

The YAML file looks like this:
<pre>
<code>
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  creationTimestamp: null
  name: project-request
<span  style="background-color: #ccd1f0">objects:</span>
- apiVersion: project.openshift.io/v1
  kind: Project
  metadata:
    annotations:
      openshift.io/description: ${PROJECT_DESCRIPTION}
      openshift.io/display-name: ${PROJECT_DISPLAYNAME}
      openshift.io/requester: ${PROJECT_REQUESTING_USER}
    creationTimestamp: null
    name: ${PROJECT_NAME}
  spec: {}
  status: {}
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    creationTimestamp: null
    name: admin
    namespace: ${PROJECT_NAME}
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: admin
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: ${PROJECT_ADMIN_USER}
<span  style="background-color: #ccd1f0">parameters:</span>
- name: PROJECT_NAME
- name: PROJECT_DISPLAYNAME
- name: PROJECT_DESCRIPTION
- name: PROJECT_ADMIN_USER
- name: PROJECT_REQUESTING_USER
</code>
</pre>

!!! note
    When a user requests a project, OpenShift replaces the `${VARIABLE}` syntax with the parameters of the project request, and creates the objects in the objects key.

    - `objects` contains a list of resources to be created
    - `parameters` contains a list of available parameters



Some resource in the **project template** such as `quotas` - do not have strict validation. -》 即使有错也不会报错！！比如把`count/pod`写成了`count/pods`。以防万一，可以这样做：

1. 创建一个namespace
2. 创建 resources 比如 `RoleBinding` 直到获得预期的行为。
3. 将 resources 用 YAML 打印出来，清理掉没必要的部分比如`status`
4. 将 resources 加入 `oc adm create-bootstrap-project-template -o yaml > file` 文件
5. 再生成 **project template**: 
```bash
oc create -f template -n openshift-config
```

### Usage
Now we have created the **project template**, to use it, we update the `project` resource:

<pre>
<code>
apiVersion: config.openshift.io/v1
kind: Project
metadata:
...output omitted...
  name: cluster
...output omitted...
<span  style="background-color: #ccd1f0">
spec:
  projectRequestTemplate:
    name: project-request
</span>
</code>
</pre>

!!! note "Reverse the usage"
    To revert to the original **project template**, modify the `project` resource to clear the spec resource to match the `spec: {}` format using:

    ```bash
    oc edit projects.config.openshift.io cluster
    ```

    查看pod的重启：
    ```bash
    watch oc get pod -n openshift-apiserver
    ```

# 5. Self-provisioner `role`

- `user` with the **self-provisioner** `clusterrole` can create projects
- By default, the **self-provisioner** role is bound to all authenticated `user`


!!! warning
    Remember that `user` with `namespace` permissions can create `namespace` that do not use the **project template**.


查看该role的rolebinding：
```bash
[user@host ~]$ oc describe clusterrolebinding.rbac self-provisioners

Name:         self-provisioners
Labels:       <none>
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  self-provisioner
Subjects:
  Kind   Name                        Namespace
  ----   ----                        ---------
  Group  system:authenticated:oauth
```

!!! warning
    `Annotations:  rbac.authorization.kubernetes.io/autoupdate: true`确保 `self-provisioners` 这样的 `ClusterRoleBinding` 会在集群更新时自动同步到最新的系统策略，保持集群安全和策略一致性。

## 修改
to disable self-provisioning, using bash:
```bash
oc annotate clusterrolebinding/self-provisioners \
  --overwrite rbac.authorization.kubernetes.io/autoupdate=false

oc patch clusterrolebinding.rbac self-provisioners \
  -p '{"subjects": null}'
```

or edit YAML:
```bash
oc edit clusterrolebinding/self-provisioners
```