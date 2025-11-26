if we do `stanctl air-gapped package` we will get an air-gapped zip file `instana-airgapped.tar.gz`. After we install it with the following CLIs:

```bash
stanctl air-gapped import --file instana-airgapped.tar.gz
stanctl up --air-gapped
```

There will be a `.stanctl` folder(around 140MB) after the installation:
```bash
.stanctl
├──charts/      # Helm Charts for necessary Resources
├──instana.yaml # Core Configuration File
├──k3s/         # k3s related 
├──license.json # License
├──logs/        
├──releases/    # Release info until now
├──state/       # CLI history
├──tmp/
└──values/      # Value Configuration for Resources
```


## `charts/`
This folder includes `*.zip` files of **Helm Charts** for necessary Resources:
```bash
charts
├── beeinstana-operator-v1.81.0.tgz
├── cass-operator-0.53.6.tgz
├── cert-manager-v1.17.2.tgz
├── cloudnative-pg-0.21.1.tgz
├── eck-operator-2.16.1.tgz
├── ibm-clickhouse-operator-v1.2.0.tgz
├── instana-agent-2.0.19.tgz
├── instana-beeinstana-1.2.0.tgz
├── instana-cassandra-1.2.0.tgz
├── instana-clickhouse-1.8.0.tgz
├── instana-core-1.8.1.tgz
├── instana-elasticsearch-1.2.0.tgz
├── instana-enterprise-operator-1.3.0.tgz
├── instana-kafka-1.2.0.tgz
├── instana-local-path-provisioner-1.1.0.tgz
├── instana-postgres-1.3.0.tgz
├── instana-unit-1.0.0.tgz
└── strimzi-kafka-operator-helm-3-chart-0.45.0.tgz
```

unzip some of them as examples:

### beeinstana-operator/
This chart will install the beeinstana operator into a namespace. It includes `Deployment`, `ClusterRole`, `ClusterRoleBinding`, `ServiceAccount`

```bash
beeinstana-operator
├── Chart.yaml
├── README.md
├── templates
│   ├── _helpers.tpl
│   ├── crds
│   │   └── beeinstana.instana.com_beeinstanas.yaml
│   ├── deployment-operator.yaml
│   ├── role-operator.yaml
│   ├── rolebinding-operator.yaml
│   └── serviceaccount-operator.yaml
└── values.yaml
```

### instana-beeinstana/
A Helm chart for Beeinstana. It includes `BeeInstana`, 2 `Secret`

```bash
instana-beeinstana
├── Chart.yaml
├── README.md
├── templates
│   ├── _helpers.tpl
│   ├── beeinstana-kafka-secret.yaml
│   ├── beeinstana-secret.yaml
│   └── beeinstana.yaml
└── values.yaml
```

### instana-core/
A Helm chart for setting up an Instana core. It includes `Core`, 3 `Secret`, `Certificate`, `Service`, `Issuer`

```bash
instana-core
├── Chart.yaml
├── README.md
├── templates
│   ├── _helpers.tpl
│   ├── core.yaml
│   ├── instana-internal-secret.yaml
│   ├── issuer.yaml
│   ├── secret.yaml
│   ├── self-signed-ca-secret.yaml
│   ├── services.yaml
│   └── tls-secret.yaml
└── values.yaml
```

### instana-unit/
A Helm chart for setting up an Instana unit. It includes `Unit`, `Secret`
```bash
instana-unit
├── Chart.yaml
├── README.md
├── templates
│   ├── _helpers.tpl
│   ├── secret.yaml
│   └── unit.yaml
└── values.yaml
```

## `instana.yaml`
This is the <u>Core Configuration File</u> that includes configs for:

- **Meta info**: Instana version, type(demo or production), download key, sales key, Agent version
- **Datastore credentials**: `kafka`, `postgres`, `clickhouse`, `elasticsearch`, `cassandra`, `beeinstana`
- **Storage config**: directories for 4 essential volumnes: `volume-data`, `volume-metrics`, `volume-analytics`, `volume-objects`
- **Instana Artifactory**: `https://artifact-public.instana.io`
    - k3s image: `https://artifact-public.instana.io/artifactory/rel-generic-instana-virtual/k3s-io/k3s/releases/download`
    - `charts/`中的 Resources Helm Charts: `https://artifact-public.instana.io/artifactory/rel-helm-customer-virtual`

!!! note "Instana Artifactory"
    to check whats inside in **Instana Artifactory**:

    ```bash
    curl -u [user]:[password] https://artifact-public.instana.io/v2/_catalog
    ```

## `k3s/`
This includes k3s rancher stuffs:
```bash
k3s
├── k3s                     # binary
├── k3s-install.sh          # script to install k3s
├── k3s-pre-install.sh      # script to prepare k3s installation
├── kubelet-config.yaml     # KubeletConfiguration
└── registries.yaml
```

## `license.json`
```bash
logs
└── console.log
```

## `logs/`

## `releases/`
it contains all the releases until now
```bash
releases
├── custom-edition-releases.yaml
└── standard-edition-releases.yaml
```

!!! note "standard-edition-releases.yaml"
    ```yaml
    releases:
    - version: 1.6.0
        minSupported: 1.0.0
        maxSupported: 1.1.1
    ```

## `state/`
it contains CLI history

```bash
state
└── .state.yaml
```

## `tmp/`

## `values/`
it contains value configuration for all resources: 

- operator version
- images version
- container resource: cpu, memory
```bash
values
├── beeinstana
│   └── instana-values.yaml
├── beeinstana-operator
│   └── instana-values.yaml
├── cass-operator
│   └── instana-values.yaml
├── cassandra
│   └── instana-values.yaml
├── cert-manager
│   └── instana-values.yaml
├── clickhouse
│   └── instana-values.yaml
├── cloudnative-pg
│   └── instana-values.yaml
├── contour
│   └── instana-values.yaml
├── eck-operator
│   └── instana-values.yaml
├── elasticsearch
│   └── instana-values.yaml
├── ibm-clickhouse-operator
│   └── instana-values.yaml
├── instana-beeinstana
│   └── instana-values.yaml
├── instana-cassandra
│   └── instana-values.yaml
├── instana-clickhouse
│   └── instana-values.yaml
├── instana-core
│   └── instana-values.yaml
├── instana-elasticsearch
│   └── instana-values.yaml
├── instana-enterprise-operator
│   └── instana-values.yaml
├── instana-kafka
│   └── instana-values.yaml
├── instana-local-path-provisioner
│   └── instana-values.yaml
├── instana-postgres
│   └── instana-values.yaml
├── instana-registry
│   └── instana-values.yaml
├── instana-unit
│   └── instana-values.yaml
├── kafka
│   └── instana-values.yaml
├── local-path-provisioner
│   └── instana-values.yaml
├── postgres
│   └── instana-values.yaml
└── strimzi-kafka-operator
    └── instana-values.yaml
```