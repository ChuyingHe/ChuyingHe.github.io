- [Doc](https://www.ibm.com/docs/en/instana-observability/1.0.308?topic=installing-instana-backend)

# Prerequisites
1. kubectl plugin: `kubectl instana` : tool to generate YAML for **namespaces** and **custom resources**
2. data store
3. Instana Enterprise Operator


# Preparation
1. create **namespaces**
1. create **image pull secrets**
1. download the license file using `kubectl instana`
1. create other **secrets** 
    - `instana-tls` to hold certificate
    - `instana-core`(in NS `instana-core`) to hold a `config.yaml` file - this includes all the Instana configurations such as `dhParams`(Diffie-Hellman parameter), `storageConfigs`, `datastoreConfigs` etc
    - `tenant0-unit0`(in NS `instana-units`) to hold a `config-unit.yaml` file
1. create **custom resources** Core (named `instana-core`) in manifest:
    ```yaml
    apiVersion: instana.io/v1beta2
    kind: Core
    metadata:
        namespace: instana-core
        name: instana-core
    spec:
        # resourceProfile -> CPU/Memory
        # agentAcceptorConfig -> host, port. The acceptor is the endpoint that Instana agents need to reach to deliver traces or metrics to the Instana backend.
        # gatewayConfig: ingress, port, loadbalancer, tls-termination
        # storageConfigs -> for "Raw Spans Data" -> choose from S3, GCS, Azure or FileSystem
        # datastoreConfigs -> for "data store", must include:
            # cassandraConfigs:
            # clickhouseConfigs:
            # postgresConfigs:
            # elasticsearchConfig:
            # kafkaConfig:  
        # properties -> extra properties such as "config.synthetics.retention.days"
    ```

    !!! note "Raw Spans Data"
        **原始跨度数据** 指的是：在分布式系统中，由各个服务实例实时产生的、记录了请求处理细节的原始、未加工、高保真的追踪数据集合。它包含了构成一条完整请求链（Trace）所需的所有 Span 的详细信息。

    !!! note "data store"
        find the doc [here](https://www.ibm.com/docs/en/instana-observability/1.0.308?topic=installing-instana-backend#data-stores__title__1).

        If data stores are installed <u>in the same cluster</u> by using third-party Operators, you can configure `datastoreConfigs` to connect to the data stores by using **in-cluster DNS hostnames** 

1. create **custom resources** Unit (named `tenant0-unit0`) in manifest:

