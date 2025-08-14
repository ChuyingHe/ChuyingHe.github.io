在 OpenShift 4 中，Monitoring Stack 是由多个组件组成的集成监控系统，用于收集、存储和可视化集群及应用的指标，并提供告警功能。以下是核心组件及其作用：

# Prometheus

核心角色：开源监控与告警工具，负责指标收集、存储和告警评估。More info check [here](../../monitoring/prometheus/index.md)

功能：

- 定期从目标（如 Pod、节点、API Server）拉取指标（scrape metrics）。
- 存储时间序列数据，支持 PromQL 查询语言。
- 根据规则（PrometheusRule CRD）触发告警，并将告警发送至 Alertmanager。


OpenShift 集成：

- 默认部署为集群内部 Prometheus 实例（非高可用）。
- 通过 ClusterMonitoringOperator 管理生命周期。



## PrometheusRule
- 开发者可通过 Prometheus Client Library 在应用中暴露指标（如 /metrics 端点）。
- 使用 ServiceMonitor 或 PodMonitor 将指标纳入监控。

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: custom-alerts
spec:
  groups:
  - name: example
    rules:
    - alert: HighRequestLatency
      expr: http_request_duration_seconds{job="my-app"} > 1
```


# Alertmanager

核心角色：处理 Prometheus 发送的告警，进行去重、分组并路由到通知渠道（如 Email、Slack）。

功能：

- 抑制重复告警（deduplication）。
- 按标签分组告警，减少通知噪音。
- 支持静默（silence）特定告警。

OpenShift 配置：

- 通过 alertmanager-main Pod 运行。
- 配置通过 Secret（alertmanager-main）管理。



# Grafana

核心角色：可视化仪表板工具，用于展示监控数据。

功能：

- 提供预置的 OpenShift 仪表板（如集群资源使用率、Pod 状态）。
- 支持自定义仪表板（需通过 ConfigMap 或 Operator 导入）。

注意：

- OpenShift 默认不暴露 Grafana，需通过路由（Route）手动访问。
- 数据源自动配置为集群内 Prometheus。


# OpenShift Monitoring Operators

关键 Operator：

1. Cluster Monitoring Operator (CMO)：
    - 管理监控组件的部署、升级和配置（Prometheus、Alertmanager、Grafana 等）。
    - 通过 cluster-monitoring-config ConfigMap 配置全局监控。
2. Prometheus Operator：
    - 自动化 Prometheus 实例的部署和管理。
    - 通过 Prometheus 和 ServiceMonitor CRD 定义监控目标。


# ServiceMonitors 和 PodMonitors

作用：动态发现需要监控的服务和 Pod。

- `ServiceMonitor`：定义如何监控一组 Service 背后的 Pod（通过 Endpoints）。
- `PodMonitor`：直接监控特定 Pod（无需通过 Service）。


```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-monitor
spec:
  endpoints:
  - port: web
  selector:
    matchLabels:
      app: my-app
```


# Node Exporter & Kube-State-Metrics

1. Node Exporter：
    - 部署在每个节点上，收集节点级指标（CPU、内存、磁盘等）。
    - 由 Prometheus 通过 ServiceMonitor/node-exporter 自动抓取。

2. Kube-State-Metrics：
    - 生成 Kubernetes 资源状态指标（如 Deployment 副本数、Pod 状态）。
    - 通过 ServiceMonitor/kube-state-metrics 被 Prometheus 抓取。


# Thanos（可选扩展）

场景：长期存储和全局视图（多集群监控）。

功能：

- 与 OpenShift 集成需手动配置，通过对象存储（如 S3）持久化指标。
- 提供跨集群查询能力。