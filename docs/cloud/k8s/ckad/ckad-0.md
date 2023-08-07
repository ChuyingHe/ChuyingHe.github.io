[TOC]

```bash
kubectl get pods <pod-name> -o custom-columns=NAME:.metadata.name,RSRC:.metadata.resourceVersion
```

```
kubectl get pods --sort-by=.metadata.name
```


```
kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always
```
选择器可以是：
|数据对象  | 解释 |可用属性|
|--|--|--|
| `metadata` |  |`name` <br/> `namespace`|
| `status` |  |`phase` |
| `spec` |  |该资源类型中spec下的所有属性，如果你想看当前资源（比如Pod）有哪些属性，可用通过`k get pod/xxx -o yaml` |


```bash
oc get route --field-selector spec.to.name=${version}-frontend -o custom-columns=URL:.spec.host --no-headers
```

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app.kubernetes.io/managed-by: argocd
  name: argocd-server
  namespace: my-project
  uid: 7d4bssd45d
spec:
  host: argocd-server-b66.xxx.cloud
  port:
    targetPort: https
  tls:
    insecureEdgeTerminationPolicy: Redirect
    termination: passthrough
  to:
    kind: Service
    name: argocd-server
    weight: 100
  wildcardPolicy: None
```
