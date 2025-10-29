
- [Prometheus' Alertmanager Doc](https://prometheus.io/docs/alerting/latest/overview/)
- [Source code in Git](https://github.com/prometheus)

# Functionalities
1. **Manage alerts** in the configuration file: 
    - **grouping**: 决定"哪些告警"会被合并到"同一个通知"中
        - Grouping of **alerts**, timing for the grouped **notifications**, and the receivers of those **notifications** are configured by a **routing tree** 
        ```yaml
        # in Configuration File - alertmanager.yml
        group_by: ['label1'， 'label2'， ...]   # 所有标签的值一样的alerts将被归为同一组
        group_wait: 5m                         # 当第一个告警进入组时，等待一段时间再发送通知。这样可以把在等待期间内产生的、属于同一组的其他告警一并发送
        group_interval: 5s                     # 变化等待时间
        ```
    - **inhibition**: suppressing notifications for certain **alerts** if certain other alerts are already firing.
        - Inhibition 的核心思想是：当某个更高级别、更根本的告警发生时，自动静音或隐藏其他相关的、低级别的告警。
        ```yaml
        # in Configuration File - alertmanager.yml
        inhibit_rules:
            # 规则：当某个节点宕机时，抑制所有源自该节点的其他告警
            - source_matchers:
                - alertname = "NodeDown"
                - severity = "critical"
                target_matchers:
            - severity =~ "warning|critical" # 抑制所有严重级别的告警
            equal:
            - node # 关键！只有当源和目标告警的 'node' 标签值相同时才抑制
        ```
    - **silences**: Silences are a straightforward way to simply mute alerts for a given time - configured based on <u>matchers</u>, just like the **routing tree** aggregation,  etc
        - in the Alertmanager UI! not Configuration File
1. **Send alerts**: per mail, slack, telegram etc

# Configurations
Example: 
```yaml
global:
    [ smtp_from: <tmpl_string> ]
    [ smtp_smarthost: <string> ]

templates:
  [ - <filepath> ... ]

route: <route> 

receivers:
  - <receiver> ...

inhibit_rules:
  [ - <inhibit_rule> ... ]

time_intervals:
  [ - <time_interval> ... ]
```

!!! warning "Verification"
    use the `amtool` for Alertmanager Configuration Grammar Verification:
    ```bash
    amtool check-config config.yml
    ```

!!! info "Data type"
    - `<duration>`: `1d`, `1h30m`, `5m`, `10s`
    - `<labelname>`
    - `<labelvalue>`
    - `<filepath>`
    - `<boolean>`
    - `<string>`
    - `<secret>`
    - `<tmpl_string>`: 即带variable的string，比如：
        ```yaml
        smtp_from: 'Alertmanager on {{ .GroupLabels.cluster }} <alerts@company.com>'
        ```
    - `<tmpl_secret>`
    - `<int>`
    - `<regex>`

## `<route>`
!!! info "`continue`"
    - `continue: false`: it stops after the first matching child. 
    - `continue: true` the alert will continue matching against subsequent siblings.

## `<inhibit_rule>`
Both target and source alerts must have the <u>same label values for the label names</u> in the equal list.