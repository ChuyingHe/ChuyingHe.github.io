# 1. Security Context Constraints
Red Hat OpenShift provides **security context constraints (SCCs)**, a security mechanism that limits the access from a running pod in OpenShift to the host environment. Cluster administrators can list the SCCs that OpenShift defines:

```bash
oc get scc
```

OpenShift provides the some default SCCs such as `anyuid` and `restricted-v2`. To get detail of one scc:

```bash
oc describe scc anyuid
```

Most pods that OpenShift creates use the `restricted-v2` SCC, which provides limited access to resources that are external to OpenShift.To view the **scc** that a pod uses:

```bash
oc describe pod console-5df4fcbb47-67c52 -n openshift-console | grep scc
```

!!! info "Use cases: when to switch SCC"
    - A container image(from DockerHub) that requires running as a specific user ID can fail because the `restricted-v2` SCC runs the container by using a random user ID.
    - A container image that listens on port 80 or on port 443 can fail for a related reason. The random user ID that the `restricted-v2` SCC uses cannot start a service that listens on a **privileged network port** (port numbers that are less than 1024)

    To debug: use the `scc-subject-review` subcommand to list all the security context constraints that can overcome the limitations that hinder the container: 查看当前deployment使用的scc <br/>
    ```bash
    oc get deployment deployment-name -o yaml | oc adm policy scc-subject-review -f -
    ```

!!! info "`oc scc-subject-review`"

    ```bash
      # Check whether user bob can create a pod specified in myresource.yaml
      oc policy scc-subject-review -u bob -f myresource.yaml

      # Check whether user bob who belongs to projectAdmin group can create a pod specified in myresource.yaml
      oc policy scc-subject-review -u bob -g projectAdmin -f myresource.yaml

      # Check whether a ServiceAccount specified in the pod.template.spec in myresourcewithsa.yaml can create the pod
      oc policy scc-subject-review -f myresourcewithsa.yaml
    ```

## Switch SCC
1. create `serviceaccount`
    ```bash
    oc create serviceaccount my-sa
    ```

2. associate `serviceaccount` with an SCC
    ```bash
    #  Identify a serviceaccount by using the -z option
    oc adm policy add-scc-to-user <SCC-NAME> -z my-sa
    ```
    PS: if assign SCC to a User, just do directly without `-z` flag: `oc adm policy add-scc-to-user restricted user1 user2`

3. Change an existing deployment to use the `serviceaccount`
    ```bash
    oc set serviceaccount deployment/deployment-name my-sa
    ```



# 2. Access to Kubernetes APIs
With the Kubernetes APIs, a user or an application can query and modify the cluster state. To protect your cluster from malicious interactions, you must grant access to the different Kubernetes APIs. **Role-based access control (RBAC)** authorization is preconfigured in OpenShift. An application requires explicit RBAC authorization to access restricted Kubernetes APIs.

!!! note "serviceaccount"
    A `serviceaccount` is a Kubernetes object within a project. The service account represents the identity of an application that runs in a pod. To grant an application access to a Kubernetes API, take these actions:

    1. Create an application service account.
    2. Grant the service account access to the Kubernetes API.
    3. Assign the service account to the application pods.

    ⚠️ If the pod definition does not specify a service account, then the pod uses the `default` service account. 


## Who needs extra access to k8s API?
For regular workloads, `default` service account is enough, but in some "infrastructure cases" we might need extra access to monitor/modify cluster resources:

- **Monitoring Applications**: need read access to watch cluster resources - such as **Red Hat Advanced Cluster Security (ACS)**
- **Controllers**: Controllers are applications that constantly watch and try to reach the intended state of a resource  - such as **ArgoCD**
- **Operators**: Operators automate creating, configuring, and managing instances of Kubernetes-native applications. Therefore, operators need permissions for configuration and maintenance tasks - such as a **database operator** 


## How to get the correct access to k8s API?
1. create role/clusterrole:
    写yaml文件
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: secret-reader             # role name
    rules:
    - apiGroups: [""]                 # The API groups, where an empty string represents the core API
      resources: ["secrets"]          # The resources that the role refers to
      verbs: ["get", "watch", "list"] # The verbs/actions that the role allows the application to do
    ```
    或者用命令行：
    ```bash
    oc create clusterrole secret-reader --verb=get,watch,list --resource=secret
    ```
2. bind role to serviceaccount
    ```bash
    # role
    oc adm policy add-role-to-user <RoleName> -z service-account

    # clusterrole
    oc adm policy add-cluster-role-to-user <ClusterRoleName> -z service-account
    ```
3. assign `serviceaccount` to `pod`


!!! note "Accessing API Resources in a Different Namespace"
    **Request:** Pod in `project-1` wants to read Secret in `project-2`
    
    **Background:** you have an application pod in the `project-1` project that requires access to `project-2` secrets, then you must take these actions:

    1. Create an `app-sa` service account in the `project-1` project.
    2. Assign the `app-sa` service account to your application pod.
    3. Create a role binding on the `project-2` project that references the `app-sa` service account and the secret-reader role or cluster role. 
        ```bash
        # 如何引用另一个 project 中的 sa:
        system:serviceaccount:<namespace>:<serviceaccount-name>
        ```
        create role-binding:
        ```bash
        oc project project-2
        oc adm policy add-role-to-user edit system:serviceaccount:project-1:my-sa
        ```

    <img src="../imgs/appsec-api-role-binding.svg" />


# 3. Job & CronJob
Basic knowledge about `job` and `cronjob` check [here](../../../k8s/ckad/ckad-5/#job). One of the use case of `cronjob` is to schedule **maintenance tasks in the cluster**.


!!! note
    Cluster maintenance tasks require privileged pods, whereas most applications might not require elevated privileges.


Job with **dry run**:

```bash
oc create job --dry-run=client -o yaml test \
    --image=registry.access.redhat.com/ubi8/ubi:8.6 \
    -- curl https://example.com
```

CronJob with **dry run**:

```bash
oc create cronjob --dry-run=client -o yaml test \
  --image=registry.access.redhat.com/ubi8/ubi:8.6 \
  --schedule='0 0 * * *' \
  -- curl https://example.com
```

!!! info "Schedule specificiation in CronJob"
    The schedule specification for Kubernetes cron jobs is derived from the specification in Linux cron job

    ```bash
    # Example cron job definition:
    # ┌───────────────── minute (0 - 59)
    # │  ┌────────────── hour (0 - 23)
    # │  │   ┌────────── day of month (1 - 31)
    # │  │   │   ┌────── month (1 - 12) or jan,feb,mar,apr ...
    # │  │   │   │   ┌── day of week (0 - 6) or sun,mon,tue,wed,thu,fri,sat
    # │  │   │   │   │    
    # m  h  dom mon dow  command
      0 */2  *   *   *   /path/to/task_executable arguments
    ```

    ⚠️ "dow"中，Sunday 是 0


    Examples:

    - `0 0 * * *`:	Run the specified task every day at midnight
    - `0 0 * * 7`:	Run the specified task every Sunday at midnight
    - `0 */4 * * *`:	Run the specified task every four hours
    - `0 * * * *`:	Run the specified task every hour
        - `0` means the job will run at the 0th minute (the start of the hour)
        - `*` runs every hour
        - `*` runs every day of month (dom) 
        - `*` runs every month (mon) 
        - `*` runs every day of week (dow) 
    
## Cronjob with raw script
example:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: wordpress-backup
spec:
  schedule: 0 2 * * 7  
  jobTemplate:  
    spec:
      template: 
        spec:  
          dnsPolicy: ClusterFirst
          restartPolicy: Never
          containers: 
          - name: wp-cli
            image: registry.io/wp-maintenance/wp-cli:2.7 
            resources: {}
            command:  
            - bash
            - -xc
            args:  
            - >
              wp maintenance-mode activate ;
              wp db export | gzip > database.sql.gz ;
              wp maintenance-mode deactivate ;
              rclone copy database.sql.gz s3://bucket/backups/ ;
              rm -v database.sql.gz ;
```

!!! note 
    The `>` symbol uses the **YAML folded style**, which converts all newlines to spaces when parsing. Each command is separated with a semicolon (`;`), because the string in the `args` key is passed as a single argument to the `bash -xc` command.

    the yaml code above is equivalent to:

    ```bash
    bash -xc 'wp maintenance-mode activate ; wp db export | gzip > database.sql.gz ; wp maintenance-mode deactivate ; rclone copy database.sql.gz s3://bucket/backups/ ; rm -v database.sql.gz ;'
    ```

!!! note "`/bin/sh -c` VS `bash -xc`"
    |Feature | `/bin/sh -c`| `bash -xc`|
    |:-|:-|:-|
    |Flags|`-c` execute|`-cx` execute with debug|
    |Type | Lightweight shell (POSIX-compliant) | Advanced shell with extra features|
    |Location | Usually /bin/sh (may point to different shells) | Usually /bin/bash|
    |Scripting | Supports basic scripting | Supports advanced scripting (arrays, associative arrays, string manipulation, etc.)|
    |Command History (history) | Not always available | Fully supported|
    |Tab Completion | Minimal or none | Fully supported|
    |Loops & Conditionals | POSIX standard | Extended syntax ([[ ... ]], == operator)|
    |Process Substitution (<(cmd)) | Not supported | Supported|
    |Array Variables (arr=(a b c)) | Not supported | Supported|


## Cronjob with ConfigMap's script
1. create `configmap` with the required script:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: maintenance
  app: crictl
data:
  maintenance.sh: |
    #!/bin/bash
    NODES=$(oc get nodes -o=name) 
    for NODE in ${NODES} 
    do
      echo ${NODE}
      oc debug ${NODE} -- \ 
        chroot /host \
          /bin/bash -xc 'crictl images ; crictl rmi --prune' 
      echo $?
    done
```
2. mount the `configmap` to a `cronjob`

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: image-pruner
spec:
  schedule: 0 * * * *
  jobTemplate:
    spec:
      template:
        spec:
          dnsPolicy: ClusterFirst
          restartPolicy: Never
          containers:
          - name: image-pruner
            image: quay.io/openshift/origin-cli:4.14
            resources: {}
            command:
            - /opt/maintenance.sh
            # Mounting the configuration map as a volume
            volumeMounts:
            - name: scripts
              mountPath: /opt
          volumes:
          - name: scripts
            configMap:
              name: maintenance
              defaultMode: 0555
```

# Exercies notes


```bash
oc debug node/master01 -- \   # Starts a debug pod on the node master01.
    chroot /host \            # Changes Root-Filesystem to /host, which gives access to the node’s actual file system.
    crictl rmi --prune        # Uses crictl (CRI-O CLI) to remove all unused images from the node.
```

!!! note "crictl"
    Directly interacts with the container runtime (`CRI-O`), allowing you to manage containers and images at the node level.