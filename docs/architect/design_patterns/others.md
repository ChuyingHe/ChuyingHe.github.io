# hub-and-spoke Architectual Pattern
“Hub-and-Spoke 架构模式”是一种常用于网络、云基础架构和系统集成中的架构设计模式。它模仿了 **轮毂和辐条（hub-and-spoke）**的结构：

一个中心节点（Hub）连接多个外围节点（Spokes），而外围节点之间通常不直接通信。

```less
           Spoke A
              |
              |
Spoke B ——— Hub ——— Spoke C
              |
              |
           Spoke D

```

- Hub（中心）：负责协调、管理、路由或控制整个系统。
- Spokes（辐条）：外围节点，执行具体的功能或服务，只与 Hub 通信。


🧰 2. 应用场景举例

|场景|描述|
|--|--|
|网络拓扑|云中常见的网络设计：中心虚拟网络（Hub VNet/VPC）连接多个子网（Spokes）|
微服务集成 使用消息总线（Hub）连接多个微服务（Spokes），实现解耦
|数据集成|中心数据平台聚合多个来源的数据|
|多租户云架构|一个中央管理平台为多个租户提供服务支持|