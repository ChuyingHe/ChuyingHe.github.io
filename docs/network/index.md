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


