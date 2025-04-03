当前受欢迎的负载均衡器（Load Balancer）：

| **Load Balancer**        | **Features**                                              | **Use Cases**                                   |
|-------------------------|---------------------------------------------------------|-----------------------------------------------|
| **HAProxy**             | Open-source, high-performance, supports TCP/HTTP LB and reverse proxy | Web applications, Kubernetes, OpenShift, microservices |
| **NGINX**               | Acts as a web server, reverse proxy, and load balancer, supports HTTP/HTTPS, TCP/UDP | High-traffic websites, API gateways, microservices |
| **Envoy**               | Modern proxy, supports gRPC, HTTP/2, dynamic service discovery | Kubernetes, microservices, service mesh (Istio) |
| **F5 BIG-IP**           | Hardware/software combo, enterprise-grade LB, advanced security features | Large enterprises, banks, data centers |
| **AWS Elastic Load Balancer (ELB)** | Cloud-native LB, supports ALB (Application), NLB (Network), CLB (Classic) | AWS cloud applications, auto-scaling, high availability |

# HAProxy

haproxy.cfg 是 HAProxy 的核心配置文件，用于定义 负载均衡和代理规则。在 OpenShift 和其他分布式系统中，HAProxy 通常用于 流量分发 和 高可用性（HA），确保多个后端服务器之间负载均衡，并提供健康检查、SSL 终结等功能。

配置结构

一个典型的 haproxy.cfg 文件包含多个部分，主要包括：

## 全局设置（global）
##  默认设置（defaults）


## 前端（frontend）
```ini
frontend http_front         #  定义前端
bind *:80                   #  监听 80 端口
default_backend http_back   #  默认转发到 http_back 这个后端
```

!!! info 
    在 haproxy.cfg 配置文件中，前端（frontend） 和 后端（backend） 是 请求流量的入口和出口，用于定义 负载均衡规则。

    - 前端（frontend） → 负责接收请求，决定如何路由
    - 后端（backend） → 负责处理请求，执行负载均衡

## 后端（backend）
```ini
backend http_back                   # 定义后端服务器池
balance roundrobin                  # 轮询调度（leastconn → 选择连接最少的服务器, source → 根据客户端 IP 进行会话保持）
server web1 192.168.1.101:80 check  # 指定后端服务器，并开启健康检查
server web2 192.168.1.102:80 check  # 备用服务器，可以有多个
```
## 监听（listen）（可选）
```ini
listen stats                    # 监听管理页面
   bind *:9000                  # 监听 9000 端口
   stats enable             
   stats uri /haproxy?stats     # 用于查看 statistics 的URL
   stats auth admin:password    # 认证信息
```
