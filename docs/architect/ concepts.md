# Serverless
“Serverless”（无服务器）这个词在云计算中常常会引起误解。它并不是真的“没有服务器”，而是指开发者不需要管理服务器。下面是对这个词的详细解释：

📌 Serverless 的真正含义：

不是没有服务器：实际上，Serverless 架构仍然依赖服务器，但这些服务器是由云服务提供商（比如 AWS、Azure、Google Cloud）管理的。
开发者不需要关心服务器：你不需要配置、维护或扩展服务器。只关注代码本身。


✅ Serverless 的特点：

|特点|描述|
|--|--|
|按需运行|代码只在被触发时执行（如 HTTP 请求、数据库变更、队列事件等）|
|自动扩展|不用设置扩容，平台会根据负载自动伸缩|
|无需服务器管理 |云平台管理操作系统、打补丁、监控、资源配置等|
|计费模式|按请求和运行时间计费（不是按服务器小时数）|



# Multi-tenant

Multi-tenant service（多租户服务） 是一种软件架构模式，在这种模式下：多个客户（租户，tenants）共享同一个应用实例，但数据和配置彼此隔离。


🏢 想象这样一个场景：

你在使用一个 SaaS 服务（比如 Notion、Slack 或 Zoom）。你和其他公司都在用同一个平台，但你们的数据彼此隔离、看不到对方，这就是 多租户服务的典型例子。

🧱 核心特点

|特性|	描述|
|--| --|
|共享资源|同一个应用实例服务多个租户，共享计算资源、数据库实例等|
|数据隔离|每个租户的数据彼此隔离，防止越权访问|
|统一维护|开发者只维护一套代码、部署一次，即可服务多个客户|
|配置灵活|每个租户可有自己的设置、品牌、语言、权限等个性化配置|