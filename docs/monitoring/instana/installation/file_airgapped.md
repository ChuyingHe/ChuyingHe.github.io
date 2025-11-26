By unzipping the `instana-airgapped.tar.gz` file we will get a `airgapped/` folder(around 47GB):


```bash
├──buildmeta/
├──cluster/
├──config/
├──docker/
├──helm/
├──license/
├──releases/
└──stanctl  # Executable
```


## `buildmeta/`

```bash
buildmeta
└── buildmeta.yaml
```


## `cluster/`

```bash
cluster
└── download
    └── v1.32.9+k3s1
        ├── k3s
        └── sha256sum-amd64.txt
```

## `config/`

```bash
config
└── instana.yaml
```

## `docker/`
here are the images that will be imported in to the Container Runtime. 92 `*.tar` files are here in total.

```bash
docker
└── images
    ├── agent_static---1.303.2.tar
    ├── backend_acceptor---3.307.450-0.tar
    ├── backend_accountant---3.307.450-0.tar
    ├── backend_action-ai-generation---3.307.450-0.tar
    ├── backend_action-orchestration---3.307.450-0.tar
    ...
    ├── self-hosted-images_k8s_coredns---1.12.4_v0.10.0.tar
    ├── self-hosted-images_k8s_envoy---1.35.0_v0.14.0.tar
    ├── self-hosted-images_k8s_klipper-lb---v0.4.13_v0.17.0.tar
    ├── self-hosted-images_k8s_local-path-provisioner---v0.0.32_v0.8.0.tar
    ├── self-hosted-images_k8s_metrics-server---v0.8.0_v0.8.0.tar
    └── self-hosted-images_k8s_pause---3.6_v0.8.0.tar
```

## `helm/`
This folder includes `*.zip` files of **Helm Charts** for necessary Resources. The content is ALMOST identical as the installation folder `.stanctl/charts/`:
```bash
helm
├── beeinstana-operator-v1.96.0.tgz
├── cass-operator-0.60.0.tgz
├── cert-manager-v1.18.3.tgz
├── cloudnative-pg-0.24.0.tgz
├── coredns-1.43.0.tgz          # extra compressed file
├── eck-operator-3.0.0.tgz
├── ibm-clickhouse-operator-v1.2.0.tgz
├── instana-agent-2.0.32.tgz
├── instana-beeinstana-1.3.0.tgz
├── instana-cassandra-1.6.0.tgz
├── instana-clickhouse-1.9.0.tgz
├── instana-core-1.14.0.tgz
├── instana-elasticsearch-1.3.0.tgz
├── instana-enterprise-operator-1.7.1.tgz
├── instana-kafka-1.6.0.tgz
├── instana-local-path-provisioner-1.1.0.tgz
├── instana-postgres-1.5.0.tgz
├── instana-unit-1.1.0.tgz
├── metrics-server-3.12.2.tgz   # extra compressed file
└── strimzi-kafka-operator-helm-3-chart-0.45.1.tgz
```

## `license/`

```bash
license
└── license.json
```

## `releases/`

```bash
```
