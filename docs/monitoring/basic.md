# Pillars of Monitoring/Observability
- **Logging（日志）**：就像病人的病历本。它记录了在特定时间点发生的事件，它是离散的、文本性的记录。比如:
    - `某年某月某日，病人主诉头痛`
    - `体温38.5°C`
- **Tracing（追踪）**：就像详细追踪病人一天内的全部活动路径。它关注的是一个请求（或事务）在分布式系统中流经的完整路径。比如:
    - `早上8点去门诊 -> 9点去放射科拍片 -> 10点去药房取药`
- **Metrics（指标）**：就像病人的生命体征仪表盘。它持续地、以固定频率测量和展示关键数据，它是数值型的、随时间变化的。比如:
    - `心率：75次/分`
    - `血压：120/80 mmHg`

!!! note "Four Golden Signals of Metrics"
    - **latency**: time to service a request 
    - **traffic**: requests/second 
    - **error**: error rate of requests 
    - **saturation**: fullness of a service


# Alert