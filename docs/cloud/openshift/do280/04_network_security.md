# 1. Encrypted Route
!!! note "encrypted/unencrypted Route"
    - **Encrypted routes** secures traffic to and from the pods. It support <ins>several types of transport layer security (TLS)</ins> termination to serve certificates to the client. 
    - **Unencrypted routes** are the simplest to configure, they require no key/certificates. 


!!! info "TLS & TLS Termination"
    <span style="background-color: #727bb8">[加密]</span> **TLS（Transport Layer Security，传输层安全协议）** 是一种用于在计算机网络上提供安全通信的协议，它是 SSL（Secure Sockets Layer）的后续版本。TLS 主要用于确保数据在客户端和服务器之间传输时的 保密性、完整性 和 身份验证。

    <span style="background-color: #727bb8">[解密]</span> **TLS Termination** 是指在网络通信中，通过一个设备/服务（如负载均衡器、代理服务器）解密 TLS 所加密的流量，从而使后续的服务器或服务可以处理明文数据。举例：在NGINX中配置TLS Termination：
    ```nginx
    server {
        listen 443 ssl;
        server_name example.com;

        ssl_certificate /path/to/cert.pem;
        ssl_certificate_key /path/to/key.pem;

        location / {
            proxy_pass http://backend-service;
        }
    }
    ```
    注意:

    - `listen 443 ssl` ：用443端口处理加密流量。
    - `proxy_pass http://backend-service;`：将解密后的流量转发给后端服务 `backend-service`。



## 常见的 TLS Termination 类型

| 特性 | **Edge Termination**| **Passthrough**| **Re-encryption** |
|:-|:-|:-|:-|
| **解密位置** | 中间节点（如负载均衡器、边缘代理）| 后端服务| 中间节点和后端服务均需加解密|
| **加密范围** | 客户端到中间节点 | 客户端到后端（端到端加密）| 客户端到中间节点 + 中间节点到后端 |
| **安全性** | 明文流量在中间节点和后端之间传输，需内网保障 | 流量始终加密，安全性最高| 提供更多安全控制，适合复杂场景|
| **性能消耗** | 减轻后端加解密负担，提升性能 | 中间节点性能消耗最少| 中间节点性能开销更大|
| **中间节点功能** | 可执行路由、认证、日志记录等操作| 无法查看流量内容，功能受限| 可执行路由、认证、日志记录等操作|
| **证书管理**<br/>**复杂性** | 中间节点需管理证书，后端无需管理| 仅后端管理证书，配置简单| 中间节点和后端均需管理证书|
|**适用场景**|适用于性能优先的场景，如负载均衡器需（load balancer）要处理大量 TLS 流量。因为Edge为后端减负，而且可以用load balancer来可以提升系统的智能性和管理效率|适合对安全性要求高的场景，如需要端到端 TLS 保护的应用。|适合微服务架构中，既需要中间节点控制流量，又需要保证后端加密传输的情况。|

!!! note "更多教程"
    - [HTTP, HTTPS, CA, Certificate](https://www.youtube.com/watch?v=EnY6fSng3Ew&ab_channel=LaithAcademy)
    - [Redhat Blog](https://www.redhat.com/en/blog/self-serviced-end-to-end-encryption-approaches-for-applications-deployed-in-openshift)


### Edge
<img src="../imgs/tls_edge.svg" />

### Passthrough
<img src="../imgs/tls_passthrough.svg" />

### Re-encryption

!!! note "Create route with **Edge Termination**"
    to create an encrypted edge route with a custom TLS certificate:

    ```bash
    oc create route edge \
        --service api-frontend \
        --hostname api.apps.acme.com \
        --key api.key \
        --cert api.crt
    ```

    when `--key` and `--cert` exist,  then the **RHOCP ingress operator** provides a certificate from the **internal Certificate Authority (CA)**


!!! info "Certificates provided from internal 'Certificate Authority'"
    to view the **certificates** that the internal **CA** provides:
    ```bash
    oc get secrets/router-ca -n openshift-ingress-operator -o yaml
    ```
## Example
### Create unencrypted route
```bash
oc expose svc/my_service
```

### Create encrypted route

- [Edge Termination](./04_ext_edge.md)
- [Passthrough Termination](./04_ext_passthrough.md)

# 2. NetworkPolicy (`netpol`)
NetworkPolicy（`netpol`） 是一个kubernetes的概念，详见[CKAD 6.服务与网络](../../../k8s/ckad/ckad-6/#3-networkpolicy). 

In contrast to traditional firewalls,  `netpol` network traffic between pods by using **labels** instead of **IP addresses**. 

!!! note "Example"
    1.Label the namespace `network-1` with `network=network-1`:

    ```bash
    oc label namespace network-1 network=network-1
    ```

    2.Define `netpol`:<br/>
    **Example 1** <br/>
    allows communication "all pods in NS `network-2` --> all pods in NS `network-1`" <br/>
    ⚠️ `podSelector: {}` means all Pods in current NS

    ```yaml
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
        name: network-2-policy
        namespace: network-2
    spec:
        podSelector: {}
        ingress:
        - from:
            - namespaceSelector:
                matchLabels:
                network: network-1
    ```

    **Example 2**<br/>
    blocks all traffic, because no ingress rules are defined
    ```yaml
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
        name: default-deny
    spec:
        podSelector: {}
    ```

    **Example 3**<br/>
    allows communication FROM pod `deployment: product-catalog` in NS `network-1` TO pod `role: qa` in NS `network-2`:

    ```yaml
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
    name: network-1-policy
    namespace: network-1
    spec:
        podSelector:
            matchLabels:
                deployment: product-catalog
    ingress:  
    - from:  
        - namespaceSelector:
            matchLabels:
                network: network-2
        podSelector:
            matchLabels:
                role: qa
        ports:  
        - port: 8080
        protocol: TCP
    ```

    

When you protect your pods by using network policies, Some **OpenShift cluster services** might need explicit policies to access pods. The following policies allow ingress from OpenShift <ins>monitoring and ingress</ins> pods:

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-openshift-ingress
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          policy-group.network.openshift.io/ingress: ""
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-openshift-monitoring
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          network.openshift.io/policy-group: monitoring
```


# 3. Protect Internal Traffic with TLS

By default, **OpenShift** encrypts network traffic between **Nodes** and the **Control Plane**, and prevents external entities from reading internal traffic. This encryption provides stronger security than default **Kubernetes**, which does not automatically encrypt internal traffic. 

## OC `service-ca` Controller 
`service-ca` 控制器负责为集群中的服务 **提供和管理证书**。通过自动管理证书，极大简化了服务之间的安全通信配置，减少了手动操作和潜在的配置错误。

The Controller generates and signs **service certificates** for internal traffic by creating a `secret` contains 
a <ins>signed certificate</ins> and <ins>key</ins>. --> then, a `deployment` can mount this `secret` as a volume to use the <ins>signed certificate</ins>. 

### 1. Secret generation
```bash
oc annotate service hello \
    service.beta.openshift.io/serving-cert-secret-name=hello-secret
```
With this annotation, the **`service-ca` Controller** auto-generates a `secret` that named `hello-secret`, which contains <ins>signed certificate</ins> and <ins>a TLS key</ins>.

!!! note "the auto-generated Secret `hello-secret`"
    ```bash
    [student@workstation ~]$ oc describe secret hello-secret
    Name:         server-secret
    Namespace:    network-svccerts
    ...output omitted...
    Type:  kubernetes.io/tls

    Data
    ====
    tls.key:  1675 bytes
    tls.crt:  2615 bytes
    ```

### 2. Mount secret to deployment
YAML file of a NGINX `deployment`:

```yaml
...
spec:
  template:
    spec:
      containers:
        - name: hello
          volumeMounts:
            - name: hello-volume            # volume definition
              mountPath: /etc/pki/nginx/    # the application-specific mount path
      volumes:
        - name: hello-volume                # volume definition
          secret:
            defaultMode: 420                # the read-write permissions that the application recommends
            secretName: hello-secret        # the auto-generated Secret
            items:
              - key: tls.crt                # - Secret's certificate
                path: server.crt
              - key: tls.key                # - Secret's key
                path: private/server.key
...
```


!!! note "Certificate Authority (CA)"
    In cryptography, a **certificate authority** is an entity that stores, signs, and issues digital certificates. 是负责发放和管理数字证书的**权威机构**

    <img src="../imgs/ca.png" width="200" />

### 3. Certificate validation
For a client service application to verify the validity of a certificate, the application needs the CA bundle that signed that certificate. The **`service-ca` Controller** injects the **CA bundle** when you apply the annotation `service.beta.openshift.io/inject-cabundle=true` to an object.

The object could be:

- ConfigMap
- API Service: in `spec.caBundle` field
- CRD: in `spec.conversion.webhook.clientConfig.caBundle` field
- Mutating or validating webhook: in `clientConfig.caBundle` field

Example: annotating a `ConfigMap`:

```bash
oc annotate configmap ca-bundle \
    service.beta.openshift.io/inject-cabundle=true
```


!!! note "CA Bundle"
    A **CA bundle**, or **Certificate Authority bundle**, is a group of individual **SSL certificates** that are bundled together into a single file. 

    OpenShift/Kubernetes 中的服务会通过 **CA Bundle** 确定哪些 **CA** 是可信任的，从而验证通信对方的证书，如图：

    <img src="../imgs/ca_bundle.png" width="500" />

### 4. Certificate rotation

The service CA certificate is valid for 26 months by default and is automatically rotated after 13 months. After rotation is a 13-month grace period where the original CA certificate is still valid. During this grace period, each pod that is configured to trust the original CA certificate must be restarted in some way - a restart of a `service` will automatically injects the new **CA bundle**.

You can also manually rotate the certificate: <br/>
    1. service CA's certificate<br/>
    The Service CA is the certificate authority that signs and issues certificates for services in the cluster.
    ```bash
    oc delete secret/signing-key -n openshift-service-ca
    ```
    2. **generated service certificate**
    **generated service certificate** are TLS certificates automatically created by the service-ca controller for individual services or routes.
    ```bash
    # delete the existing secret, and the service-ca controller automatically generates a new one.
    oc delete secret certificate-secret
    ```

!!! info "rotate"
    In this context, "rotate" refers to the process of automatically replacing <ins>an old service CA certificate</ins> with a new one before the old certificate expires. 

# Exercise Illustration

## 4.4 Configure Network Policies
<img src="../imgs/exercise_4.4.png" width="300" />

## 4.6 Protect Internal Traffic with TLS
<img src="../imgs/exercise_4.6.png" width="500" />