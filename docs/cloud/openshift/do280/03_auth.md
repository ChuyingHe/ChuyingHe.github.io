# 1. Users & Groups
以下这些OC Resources与 **authentication & authorization** 有关：

- `identiy`: keeps a record of successful authentication attempts from a specific user and identity provider. 
- `role`: it defines set of allowed API operations for `user`, `group` or `serviceaccount`
    - `user`: credential for external entities (`user`, `external system`), it interacts with **API server**.  
    - `serviceaccount` / `sa`: credential for `applications`, `services`
    - `group`: set of `user`

## Users

!!! info "identity"
    ```bash
    > oc get identity
    NAME                   IDP NAME   IDP USER NAME  USER NAME  USER UID
    ...                    ...        ...            admin      6126c5a9-4d18-4cdf-95f7-b16c3d3e7f24
    myusers:new_admin      myusers    new_admin      new_admin  489c7402-d318-4805-b91d-44d786a92fc1
    myusers:developer      myusers    developer      developer  8dbae772-1dd4-4242-b2b4-955b005d9022
    ```

    - `IDP NAME`: 身份提供者（Identity Provider，IDP），表示该用户通过哪个认证系统登录（如 github、ldap、htpasswd）。
    - `IDP USER NAME`: 用户在身份提供者（IDP）中的用户名，即该用户在 IDP 认证系统中的唯一标识。

!!! info "User Types"
    1. **Regular User**: they are represented as `user` resource
    2. **System User**: Many system users are created automatically when the infrastructure is defined, for example, cluster administrator (with access to everything), a per-node user, users for routers and registries, and various others. <br/>**Name convention**: start with a `system:` prefix, such as:
        - `system:admin`
        - `system:openshift-registry`
        - `system:node:node1.example.com`

<img src="../imgs/user_sa.png" width="300" />


!!! warning "`User` vs `ServiceAccount`"
    ⚠️ `serviceaccount` can be considered as ONE TYPE of `user`

    ||`User`|`ServiceAccount`|
    |:-|:-|:-|
    |**使用者**|一个人，或一个外部系统（例如 CI/CD 工具）|应用程序，或服务|
    |**管理方式**|- 用户的认证信息存储在 <ins>外部系统</ins>（如身份提供商），而不是 OpenShift 内部。<br/>- OpenShift 支持多种用户认证方式|- 是集群中的资源，可以通过 YAML 或 CLI 管理。<br/>- 每个命名空间都有默认的 `default` `ServiceAccount`|
    |**适用场景**|- 开发者、管理员等直接操作 OpenShift 集群的用户。<br/>- 通过 CLI、Web 控制台或 API 与集群交互。<br/>- 外部集成系统需要通过用户身份进行认证（如 OAuth）。|- 用于 Pod 的运行时身份，Pod 使用 `ServiceAccount` 与集群交互。<br/>- 自动分配 Token，用于 API 访问的身份认证。<br/>- 应用程序需要读取 ConfigMap、Secrets 等集群资源时|


!!! info "ServiceAccount"
    **ServiceAccount** one type of **System User** that associated with projects
    
    - Some created automatically during project creation
    - Admin can create more `sa` to grant extra priviledges to Wordloads
    - By default, `sa` has no role
    - **Name convention**: start with a `system:serviceaccount:[Namespace]:` prefix, such as:
        - `system:serviceaccount:default:deployer`
        - `system:serviceaccount:accounting:builder`

## Groups

```bash
# create Group
oc adm groups new lead-developers

# add User to a Group
oc adm groups add-users lead-developers user1
```


### 常见的 系统级用户组

1. 认证相关的用户组

    |组名|说明|
    |:-|:-|
    |`system:authenticated`|所有已认证用户（包括 OAuth、X.509 证书、Kubeconfig 认证的用户）|
    |`system:authenticated:oauth`|所有通过 OAuth 认证的用户（即使用 oc login 通过 OAuth 登录的用户）|
    |`system:unauthenticated`|所有未认证的用户（如匿名 API 访问）|

2. 角色相关的用户组

    |组名|说明|
    |:-|:-|
    |`system:cluster-admins`|集群管理员组，拥有 最高权限，可以管理整个集群|
    |`system:masters`|集群控制组，管理 OpenShift 控制平面（Master 节点）|
    |`system:discovery`|所有用户 默认加入，可访问 oc get 相关的公开 API|
    |`system:scope-impersonation`|允许用户模拟 OAuth 作用域（Scope）|

3. 项目（Namespace）管理相关的用户组

    |组名|说明|
    |:-|:-|
    |`system:cluster-readers`|只读访问集群资源，不能修改|
    |`system:basic-users`|基本用户组，可以使用 oc whoami 查询自己的用户信息|
    |`system:build-strategists`|构建策略管理员，管理构建策略|
    |`system:image-builders`|允许在 OpenShift 内 构建镜像|
    |`system:image-pullers`|允许从 Registry 拉取镜像（通常绑定到 Namespace）|
    |`system:image-pushers`|允许向 Registry 推送镜像|
    |`system:registry`|OpenShift 内部 镜像仓库服务|

4. 特殊组

    |组名|说明|
    |:-|:-|
    |`system:nodes`|所有 OpenShift Worker 节点，用于 kubelet 访问 API|
    |`system:node-proxier`|OpenShift 网络代理（kube-proxy） 访问 API|
    |`system:router`|OpenShift Router 组件，负责 Ingress/Route 路由|
    |`system:deployer`|负责 部署 Pod，通常用于 DeploymentConfig|
    |`system:serviceaccounts`|所有 ServiceAccount 账户（用于 Pod 访问 API）|

# -------
# 2. Authentication

assigns the `cluster-admin` role to the student user so that the `student` user can do anything in the cluster:
```bash
oc adm policy add-cluster-role-to-user cluster-admin student
```

OpenShift API 有 2 种方法用于验证请求：

## 1. `X.509 客户端证书`
`X.509 client certificates`更为静态，通常用于已经依赖证书的安全环境中. This method uses the `kubeconfig` file, which embeds an **X.509 client certificate** that never expires. 

During installation, the **OpenShift installer** creates a unique `kubeconfig` file in the auth directory. The `kubeconfig` file contains specific details and parameters for the CLI to connect a client to the correct API server, including an X.509 certificate.

The installation logs provide the location of the kubeconfig file like this:
```bash
# log
INFO Run 'export KUBECONFIG=/root/auth/kubeconfig' to manage the cluster with 'oc'.
```

To use the `kubeconfig` file to authenticate oc commands, do this:
```bash
export KUBECONFIG=/home/user/auth/kubeconfig
```

## 2. `OAuth 访问令牌`
`OAuth access tokens`更加灵活，适用于动态、可扩展的访问控制场景. This method authenticates as the `kubeadmin` virtual user. Successful authentication grants an **OAuth access token**.

After installation completes, OpenShift creates the `kubeadmin` virtual user. The `kubeadmin` secret in the `kube-system` namespace contains the hashed password for the kubeadmin user. The kubeadmin user has cluster administrator privileges.

The installation logs provide the kubeadmin credentials like this:

```bash
# log
...output omitted...
INFO The cluster is ready when 'oc login -u kubeadmin -p shdU_trbi_6ucX_edbu_aqop'
...output omitted...
INFO Access the OpenShift web-console here:
    https://console-openshift-console.apps.ocp4.example.com
INFO Login to the console with user: kubeadmin, password: shdU_trbi_6ucX_edbu_aqop
```


!!! note
    OpenShift 支持 X.509 和 OAuth：
        - X.509 主要用于 API 访问（ServiceAccount 证书）。
        - OAuth 主要用于 用户登录（默认使用 OAuthServer）。
    
    你可以使用 oc login 命令通过两种方式认证：
    ```bash
    # 通过 OAuth 登录（默认方式）
    oc login --server=https://openshift.example.com --token=YOUR_OAUTH_TOKEN

    # 通过 X.509 证书登录（通常用于自动化）
    oc login --server=https://openshift.example.com --certificate-authority=/path/to/ca.crt
    ```

!!! note
    无论是哪种方法，都依赖于**身份提供者 / Identity Providers**来验证用户的身份。 **身份提供者（IdP）**是这两种认证方法的核心，负责验证用户的身份后，颁发认证凭证（`OAuth access tokens`或`X.509 client certificates`）


# 3. Configure Identity Providers（IdP）
身份提供者（IdP）是给user提供身份的工具。The OpenShift OAuth server can be configured to use many identity providers. The most common ones:

## 1.HTPasswd

<img src="../imgs/apache.png" width="100" />

HTPasswd is a simple authentication mechanism that uses an Apache-style `.htpasswd` file to store user credentials. It validates usernames and passwords against a secret that stores credentials that are generated by using the htpasswd command. 过程很简单，只需用htpasswd工具生成文件，用文件生成secret，再修改oauth以使用该secret即可，

!!! info "htpasswd flags"
    - `-c`: Create a new password file. This flag overwrites the existing file if it already exists.
    - `-B`:	Use **bcrypt encryption** for hashing the password. Bcrypt is a secure and computationally expensive algorithm, making it a strong choice for password hashing. <br/> If not, then use MD5 hashing algorithm for hasing
    - `-b`:	Batch mode. Allows you to pass the password directly on the command line, rather than being prompted interactively.
    - `-D` Delete user


### 1. Create HTPasswd File

```bash
# Create the htpasswd file
#  -> the password `redhat123` will be hashed with the MD5 algorithm
htpasswd -c -B -b /tmp/htpasswd student redhat123

# Add or update credentials
htpasswd -b /tmp/htpasswd student redhat1234

# delete
htpasswd -D /tmp/htpasswd student
```

!!! info "htpasswd文件示例"
    ```bash
    > cat ~/DO280/labs/auth-providers/htpasswd
    new_admin:$2y$05$qQaFbpx4hbf4uZe.SMLSduTN8uN4DNJMJ4jE5zXDA57WrTRlpu2QS
    new_developer:$apr1$S0TxtLXl$QSRfBIufYP39pKNsIg/nD1
    ```

### 2. Create(update) secret
Create(update) secret from the HTPasswd File. 注意所用的Namespace！

```bash
# Creating OC Secret with HTPasswd credential:
# ⚠️⚠️⚠️ After `--from-file`, prefix `htpasswd=` is needed!
oc create secret generic htpasswd-secret \
    --from-file htpasswd=/tmp/htpasswd \
    -n openshift-config

# Updating the OC HTPasswd Secret
oc set data secret/htpasswd-secret \
    --from-file htpasswd=/tmp/htpasswd \
    -n openshift-config
```

!!! note "extract from secret"
    ```bash
    # Extracting Secret to a file that locates under directory /tmp/
    oc extract secret/htpasswd-secret -n openshift-config \
        --to /tmp/ --confirm
    ```

!!! note "delete user"
    ```bash
    # 1. delete user data in the file
    htpasswd -D /tmp/htpasswd manager

    # 2. update the OC secret
    oc set data secret/htpasswd-secret \
        --from-file htpasswd=/tmp/htpasswd -n openshift-config

    # 3. ACTUALLY delete USER resource (named `manager` in this case)
    oc delete user manager

    # 4. ACTUALLY delete IDENTITY resources
    #  -> `my_htpasswd_provider` is the name of the `identityProviders` that we defined in the OAuth
    oc get identities | grep manager
    oc delete identity my_htpasswd_provider:manager
    ```


!!! warning "`user` VS `identity`"
    `user:identity = 1:n`
    
    Example: A `user` can authenticate through both an LDAP account and a GitHub account, resulting in two `identity` objects linked to the same `user`.

### 3. [Optional] Assign role to the user
```bash
[student@workstation ~]$ oc adm policy add-cluster-role-to-user \
                            cluster-admin new_admin

Warning: User 'new_admin' not found
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "new_admin"
```

### 4. Configuring the OAuth Custom Resource
To use the HTPasswd identity provider, the OAuth custom resource must be edited to add an entry to the `.spec.identityProviders` array using 
```bash
oc edit oauth
```

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider # name of this IdentityProvider, you can customize it
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpasswd-secret 
```

!!! info "alternative"
    也可以先把 oauth 资源保存成 yaml 文件，再进行修改。

    ```bash
    oc get oauth cluster -o yaml > oauth.yaml
    oc replace -f oauth.yaml
    ```


!!! note "`oc replace` VS `oc apply`"
    Some resources have immutable fields that apply won't let you change. For example, a deployment cannot have its selectors changed. `oc replace` can be used in these situations.

    - `oc apply` actually does a diff and updates the resource with a patch of the changes. 
    - `oc replace` will submit the full entire spec of the resource, as an atomic action.

!!! info "namespace `openshift-config`"
    `openshift-config` namespace is used to store global configuration data for the cluster, including authentication configurations. 这里，我们用来储存有用户名+密码的`secret`
   
!!! info "namespace `openshift-authentication`"
    `openshift-authentication` namespace is responsible for running the authentication services. 我们通过`oc get oauth cluster`修改了配置后，可以通过检查`openshift-authentication` 下pod是否完成重启，来确定`oauth`中的修改是否已经被应用
    
    ⚠️ 如果在`openshift-config`中的`secret`被修改，`openshift-authentication`中的pod也会被重启！



## 2.Keystone
Enables shared authentication with an OpenStack Keystone v3 server.

## 3.LDAP
Configures the LDAP identity provider to validate usernames and passwords against an LDAPv3 server, by using simple bind authentication.

## 4.GitHub or GitHub Enterprise
Configures a GitHub identity provider to validate usernames and passwords against GitHub or the GitHub Enterprise OAuth authentication server.

## 5.OpenID Connect
Integrates with an OpenID Connect identity provider by using an Authorization Code Flow.

# -------
# 4. Authorization
## Authorization with RBAC
**Role-based access control (RBAC)** is a technique for managing access to resources in a computer system. RBAC determines whether a user can perform certain actions within the cluster or project. There are 2 RBAC levels:

- **Cluster RBAC**: Roles and bindings that apply across all projects.
- **Local RBAC**: Roles and bindings that are scoped to a given project. Local role bindings can reference both cluster and local roles.

> 👉 See more illustration [here](../../k8s/ckad/ckad-8.md#3rbac) 

### RBAC Object

- **Rule**: Allowed actions for objects or groups of objects.
- **Role**: Sets of rules. Users and groups can be associated with multiple roles.
- **Binding**: Assignment of users or groups to a role.

### CLIs
Cluster-wide:

|CLI||
|:-|:-|
|`oc adm policy add-cluster-role-to-user [RoleName] [UserName]`|To change a regular user to a cluster administrator(`cluster-admin` role) <br/>即创建一个 `clusterrolebindings`|
|`oc adm policy remove-cluster-role-from-user [RoleName] [UserName]`|To change a cluster administrator(`cluster-admin` role) to a regular user <br/>即删除一个 `clusterrolebindings`|
|`oc adm policy who-can delete user`| to determine which user can perform what(`delete user` in this case)|
|`oc adm groups new [GroupName]`| to add new group to cluster|
|`oc adm groups add-users [GroupName] [UserName]`| to add user to a group|
|`$ oc adm policy remove-cluster-role-from-group [RoleName] [GroupName]`| to remove role from a group<br/>即删除一个 `clusterrolebindings`|

!!! note "rolebinding"
    `rolebinding` 可用于一下这两种情况：

    - role -- rolebinding -- group
    - role -- rolebinding -- user

Namespace-specific:

|CLI||
|:-|:-|
|`oc policy add-role-to-user [RoleName] [UserName] -n [ProjectName]`||

!!! note "role & cluster role"
    - **local role**： use `add-role-to-user`
    - **cluster role**： use `add-cluster-role-to-user`

!!! info "Default Roles"
    besides the `cluster-admin`, there are these roles:

    |Default roles|Description|
    |:-|:-|
    |`cluster-admin`|Cluster-wide (global). Users with this role have superuser access to the cluster resources. These users can perform any action on the cluster, and have full control of all projects.|
    |`admin`|Namespace-specific (local to a namespace). Users with this role can manage all project resources, including granting access to other users to access the project.|
    |`basic-user`|Users with this role have read access to the project.|
    |`cluster-status`|Users with this role can access cluster status information.|
    |`cluster-reader`|Users with this role can access or view most of the objects but cannot modify them.|
    |`edit`|Users with this role can create, change, and delete common application resources on the project, such as services and deployments. These users cannot act on management resources such as limit ranges and quotas, and cannot manage access permissions to the project.|
    |`self-provisioner`|Users with this role can create their own projects.|
    |`view`|Users with this role can view project resources, but cannot modify project resources.|



## 练习纠错记录

要求：<br/>
As the `new_admin` user, prevent users from creating projects in the cluster. --》这里要求你先以`new_admin`的user身份登陆，然后再阻止**所有用户**创建项目的能力

错误答案：<br/>
```bash
oc login -u new_admin -p new_password
oc adm policy remove-cluster-role-from-user self-provisioner new_admin
```

正确答案：<br/>
```bash
oc login -u new_admin -p new_password
oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
```

解释：正确答案确保 **所有用户** 都不能创建项目，错误答案只确保 **用户`new_admin`** 不能创建项目

- `self-provisioner` 是 OpenShift 中的一个默认clusterrole，该角色赋予用户创建新项目（即 namespace）的权限。
- `system:authenticated:oauth` 是一个包含所有通过 OAuth 认证的用户的组。换句话说，当用户通过 OAuth 登录到 OpenShift 集群时，他们会自动成为这个组的一员。
