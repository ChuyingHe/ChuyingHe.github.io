[What is a Proxy Server](https://www.youtube.com/watch?v=5cPIukqXe5w&ab_channel=PowerCertAnimatedVideos)

- act as a Middle Man between you and Internet
- Proxy 代替 User 去internet上索要数据


## Benefits

- Privacy: hide your IP address
- Speed: cache webpage on Proxy Server's centralised Database
- Saves Bandwidth
- Activity logging:
    - block certain websites
    - track user activities


## Shortage

- Proxy Server can NOT encrypt data

Therefore we needs VPN (Virtual Private Network) which encrypts the data


!!! info "in Openshift Installation Context"
    OpenShift installations often require access to container registries (`registry.redhat.io`, `quay.io`, `docker.io`). If these are blocked in a corporate network, the **proxy** ensures the nodes can reach them.

## ENV

`HTTP_PROXY`、`HTTPS_PROXY` 和 `NO_PROXY` 这些环境变量用于在命令行或程序中配置 HTTP(S) 代理，特别是在需要通过代理服务器访问外部网络时（例如，公司内部网络或云环境）。

- `HTTP_PROXY`: 适用于 HTTP 请求的代理地址。
- `HTTPS_PROXY`: 适用于 HTTPS 请求的代理地址。
- `NO_PROXY`: 指定不使用代理的 IP 或域名列表。

!!! info
    小大写都行

### 设置ENV
（1）临时设置（仅当前终端会话有效）

```bash
export HTTP_PROXY="http://username:password@proxy.example.com:8080"
export HTTPS_PROXY="http://username:password@proxy.example.com:8080"
export NO_PROXY="localhost,127.0.0.1,.example.com"
```

（2）永久生效
```bash
echo 'export HTTP_PROXY="http://proxy.example.com:8080"' >> ~/.zshrc
echo 'export HTTPS_PROXY="http://proxy.example.com:8080"' >> ~/.zshrc
echo 'export NO_PROXY="localhost,127.0.0.1,.example.com"' >> ~/.zshrc
source ~/.zshrc  # 使配置立即生效
```

（3）在特定命令中使用
```bash
HTTP_PROXY="http://proxy.example.com:8080" curl -I https://www.google.com
```

### 解除ENV
```bash
unset HTTP_PROXY HTTPS_PROXY NO_PROXY
```