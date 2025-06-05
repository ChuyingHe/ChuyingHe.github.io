# helm

for Custom Edition

```bash

# Instana Installation: third-party data store operators
helm install cass-operator -n instana-cassandra
helm install strimzi-kafka-operator -n instana-kafka
helm install elastic-operator -n instana-elasticsearch
helm install postgres-operator -n instana-postgres
helm install zookeeper-operator -n instana-zookeeper

# Instana agents: 
helm install instana-agent -n instana-agent 
# - Synthetic PoP (Point of presence) is the agent where the Synthetic tests are run.
helm install synthetic-pop -n syn
```

!!! note "Synthetic monitoring"
    Synthetic Monitoring（合成监控） 是一种主动的监控方式，它通过模拟用户行为或网络请求，定期从预定位置向系统发起访问，以测试和监控应用的可用性、性能和功能。

    与 Real User Monitoring（RUM）的区别:

    | | Synthetic Monitoring | Real User Monitoring (RUM)|
    |--|--|--|
    | 来源 | 模拟脚本 | 实际用户操作|
    | 流量 | 人工生成 | 来自真实流量|
    | 用途 | 可用性/功能/全球性能测试 | 真实用户体验分析|
    | 依赖用户流量？ | ❌ 不依赖 | ✅ 必须有真实用户流量|



# kubectl

namespace neutral CLIs:

```bash
# Get events
kubectl get event --sort-by .metadata.creationTimestamp -n [Namespace]

# Get pods
kubectl get pod -n [Namespace]
# Get log of a pod
kubectl logs pod/[PodName] -n [Namespace]
# Execute clis in a pod
kubectl exec -it pod/[PodName] -n [Namespace]


# Get ALL resources
kubectl get all -n [Namespace]

```

kubectl CLIs sorted by namespaces:

## instana-agent

```bash
kubectl edit agents.instana.io instana-agent -n instana-agent|
kubectl logs -l app.kubernetes.io/name=instana-agent -n instana-agent -c instana-agent --max-log-requests 999
kubectl logs -l app.kubernetes.io/component=k8sensor  -n instana-agent --max-log-requests 999 
```


## instana-beeinstana

```bash

kubectl edit instana-beeinstana instance -n instana-beeinstana
```

## instana-cassandra

```bash

```

## instana-clickhouse


```bash
kubectl delete ClickHouseInstallation instana -n instana-clickhouse
kubectl edit ClickHouseInstallation instana -n instana-clickhouse|

kubectl get ClickHouseInstallation instana -n instana-clickhouse
```

## instana-core
```bash
kubectl create secret generic instana-core --namespace instana-core --from-file=config.yaml

kubectl delete core instana-core -n instana-core
kubectl delete secret instana-core -n instana-core

kubectl edit core instana-core -n instana-core
kubectl edit secret instana-core -n instana-core

kubectl -n instana-core scale deployment --replicas 

kubectl describe core instana-core -n instana-core
kubectl get core instana-core -n instana-core

kubectl instana action reconcile --core instana-core -n instana-core
```


## instana-elasticsearch
```bash
kubectl edit elasticsearches instana -n instana-elasticsearch

```



## instana-kafka
```bash
kubectl edit kafka/instana -n instana-kafka

kubectl -n instana-kafka logs -l strimzi.io/cluster=instana,strimzi.io/component-type=kafka,strimzi.io/controller-name=instana-kafka
```



## instana-operator


```bash
kubectl edit deployment instana-operator -n instana-operator
kubectl -n instana-operator scale deployment instana-operator --replicas 2
```


## instana-postgres
```bash

```




## instana-unit

```bash
kubectl create secret generic tenant2-unit2 --from-file=config.yaml -n instana-unit|
kubectl edit unit bafin-apm -n instana-unit
kubectl -n instana-unit scale deployment --replicas 

kubectl describe unit tenant0-unit0 -n instana-unit
kubectl get unit tenant0-unit0 -n instana-unit
```


## instana-zookeeper
```bash
```


## instana-gitops

```bash
```






