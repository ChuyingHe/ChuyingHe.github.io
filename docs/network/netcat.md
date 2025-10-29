`nc` 是 netcat 的命令行工具，常被称为网络界的“瑞士军刀”，因为它能处理各种 TCP 和 UDP 网络连接。

example:
```bash
# 检查 192.168.1.1 的 80 端口是否开放
ip="192.168.1.1"
port="80"

if nc -z -w5 $ip $port 2>/dev/null; then
    echo "端口 $port 可达"
else
    echo "端口 $port 不可达"
fi
```