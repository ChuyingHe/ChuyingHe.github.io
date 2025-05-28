# IP
Find IP address in MacOS, in the following result there are several parts:

1. `lo` (loopback interface) - ignore this
2. Network Interface such as `ens192`, `eth0`, `enpXsY`
    - the IPv4 address is shown after `inet`
    - The `/24` means the subnet mask (255.255.255.0).


```bash
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever

2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:50:56:aa:bb:cc brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.10/24 brd 192.168.100.255 scope global dynamic ens192
       valid_lft 86300sec preferred_lft 86300sec
    inet6 fe80::250:56ff:feaa:bbcc/64 scope link 
       valid_lft forever preferred_lft forever
```


!!! note ipv4 vs ipv6
    https://www.geeksforgeeks.org/differences-between-ipv4-and-ipv6/

    ipv4: binary
    ipv6: hexadecimal


# CIDR
## CIDR blocks
A CIDR block is a group of IP addresses that share the same start and size. Big blocks have more IP addresses. Big internet groups give out large CIDR blocks to smaller groups. These smaller groups then give them to companies. If you’re at home, you get your CIDR block from your internet company.


## CIDR notation
CIDR notation shows an IP address with a number at the end. This number tells how much of the address is for the network. For example the 192.168.1.0/22 means the first 22 bits of the address are for a network. This way of writing a IP addresses helps the routers know where to send the data.


# ssh

`SSH` 就像是一把 远程控制的钥匙，可以安全地连接到另一台电脑。具体原理见[博客](https://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)。

<img src="imgs/ssh_versions_comparison.png" width=400 />



!!! note 
    SSH（Secure Shell）依赖于 TCP 来建立安全的远程连接：

    - TCP 是 运输层协议，负责可靠的数据传输，就像一条 稳定的公路。
    - SSH 是 应用层协议，使用 TCP 作为 底层传输通道，就像在公路上跑的 加密邮车。


# Gateway
The gateway is the device that routes traffic from your local network to the internet (or other networks).

```bash
ip route show
```
Example output:

```bash
default via 192.168.1.1 dev eth0
...
```
> `default via` shows the current gateway.



# /etc/hosts


# Private IP
私有IP地址范围是专门为在私有网络（如家庭、办公室或企业内部网络）中使用而预留的IP地址。这些地址不会在公共互联网上路由，因此可以在不同的私有网络中重复使用，而不会造成冲突。私有IP地址范围由Internet Assigned Numbers Authority (IANA) 定义，主要包括以下三个区块：

1. 【A类】10.0.0.0 到 10.255.255.255：这是一个A类地址范围，包含约1677万个IP地址。
    - 子网掩码通常为255.0.0.0（/8）。
    - 适用于大型网络。
2. 【B类】172.16.0.0 到 172.31.255.255：这是一个B类地址范围，包含约104万个IP地址。
    - 子网掩码通常为255.240.0.0（/12）。
    - 适用于中型网络。
3. 【C类】192.168.0.0 到 192.168.255.255：这是一个C类地址范围，包含约65536个IP地址。
    - 子网掩码通常为255.255.0.0（/16）。
    - 适用于小型网络，如家庭或小型办公室网络。

# Proxy

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



# DNS

```bash
$TTL 1W
@	IN	SOA	ns1.example.com.	root (
			2019070700	; serial
			3H		; refresh (3 hours)
			30M		; retry (30 minutes)
			2W		; expiry (2 weeks)
			1W )		; minimum (1 week)
	IN	NS	ns1.example.com.
	IN	MX 10	smtp.example.com.
;
;
ns1.example.com.		IN	A	192.168.1.5
smtp.example.com.		IN	A	192.168.1.5
;
helper.example.com.		IN	A	192.168.1.5
helper.ocp4.example.com.	IN	A	192.168.1.5
;
api.ocp4.example.com.		IN	A	192.168.1.5 
api-int.ocp4.example.com.	IN	A	192.168.1.5 
;
*.apps.ocp4.example.com.	IN	A	192.168.1.5 
;
bootstrap.ocp4.example.com.	IN	A	192.168.1.96 
;
control-plane0.ocp4.example.com.	IN	A	192.168.1.97 
control-plane1.ocp4.example.com.	IN	A	192.168.1.98 
control-plane2.ocp4.example.com.	IN	A	192.168.1.99 
;
compute0.ocp4.example.com.	IN	A	192.168.1.11 
compute1.ocp4.example.com.	IN	A	192.168.1.7 
;
;EOF
```

该文件中，DNS record 可以用简单的host名代替 to use just `helper` instead of `helper.example.com.`, but it depends on how your DNS resolution is configured.

Example of a /etc/resolv.conf file:
```bash
search example.com
nameserver 192.168.1.5
```
这种情况下可以只用 `helper`  比如：
```bash
helper		IN	A	192.168.1.5
```