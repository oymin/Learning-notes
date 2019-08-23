# HAProxy 对 MyCAT 负载均衡

对多个 MyCAT 负载均衡，并且在某一个 MyCAT 节点出现故障时，将故障节点踢出负载之外。

## 1. 操作步骤

1. 安装 HAProxy
2. 使用 Keepalived 监控 HAProxy
3. 配置 HAProxy 监控 MyCAT
4. 配置应用通过 VIP 访问 HAProxy

## 2. 安装 HAProxy

```bash
yum install haproxy -y

cd /etc/haproxy/

vi haproxy.cfg
```

配置 [haproxy.cfg](../file/haproxy.cfg) 文件，添加配置内容：

```yml
#监控 haproxy 运行状况端口
listen admin_status
    bind 0.0.0.0:48800
    stats uri /admin-status
    stats auth admin:admin

#监控 mycat 服务端口
listen allmycat_service
    bind 0.0.0.0:8096
    mode tcp
    option tcplog
    option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
    balance roundrobin
    server mycat_128 192.168.194.128:8066 check port 48700 inter 5s rise 2 fall 3
    server mycat_131 192.168.194.131:8066 check port 48700 inter 5s rise 2 fall 3

#监控 mycat 管理端口
listen allmycat_admin
    bind 0.0.0.0:8097
    mode tcp
    option tcplog
    option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
    balance roundrobin
    server mycat_128 192.168.194.128:9066 check port 48700 inter 5s rise 2 fall 3
    server mycat_131 192.168.194.131:9066 check port 48700 inter 5s rise 2 fall 3
```

## 3. 安装 xinetd

```bash
yum -y install xinetd
```

### 3.1 创建 `mycatchk` 文件

```bash
cd /etc/xinetd.d

vi mycatchk
```

`mycatchk` 文件内容：

```yml
# default: on
# description: monitor for mycat
service mycatchk
{
flags = REUSE
socket_type =stream
port = 48700
wait = no
user = root
server =/usr/local/bin/mycat_status
log_on_failure += USERID
disable = no
per_source = UNLIMITED
}
```

### 3.2 创建 mycat_status 文件

```bash
vi /usr/local/bin/mycat_status
```

```yml
#!/bin/bash
#/usr/local/mycat/bin/mycat_status.sh
# This script checks if a mycat server is healthy running on localhost. It will
# return:
# "HTTP/1.x 200 OK\r" (if mycat is running smoothly)
#
# "HTTP/1.x 503 Internal Server Error\r" (else)
mycat=`/usr/local/mycat/bin/mycat status | grep 'not running' | wc -l`
if [ "$mycat" = "0" ];
then
    /bin/echo -en "HTTP/1.1 200 OK\r\n"
    /bin/echo -en "Content-Type: text/plain\r\n"
    /bin/echo -en "Connection: close\r\n"
    /bin/echo -en "Content-Length: 40\r\n"
    /bin/echo -en "\r\n"
    /bin/echo -en "MyCAT Cluster Node is synced.\r\n"
    exit 0
else
    /bin/echo -en "HTTP/1.1 503 Service Unavailable\r\n"
    /bin/echo -en "Content-Type: text/plain\r\n"
    /bin/echo -en "Connection: close\r\n"
    /bin/echo -en "Content-Length: 44\r\n"
    /bin/echo -en "\r\n"
    /bin/echo -en "MyCAT Cluster Node is not synced.\r\n"
    exit 1
fi
```

给 `mycat_status` 脚本文件添加权限：

```bash
chmod a+x /usr/local/bin/mycat_status
```

执行脚本检测：

```bash
/usr/local/bin/mycat_status

# 200 OK mycat访问正常
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: close
Content-Length: 40

MyCAT Cluster Node is synced.
```

### 3.3 修改 /etc/services 增加配置

```bash
vi /etc/services

# 最后一行增加
mycatchk        48700/tcp               # mycatchk
```

### 3.4 启动 xinetd

```bash
cd /etc/xinetd.d

systemctl restart xinetd
```

查看端口是否启动：

```bash
netstat -nltp | grep 48700

# 启动成功
tcp6       0      0 :::48700        :::*         LISTEN      7733/xinetd
```

### 3.5 将 haproxy.cfg 配置绑定到网卡

查看当前使用的网卡是 `ens33`

```bash
ifconfig

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.194.128  netmask 255.255.255.0  broadcast 192.168.194.255
        inet6 fe80::dcb7:de99:6d21:3978  prefixlen 64  scopeid 0x20<link>
        inet6 fe80::ccbd:f58c:1b2f:8ad3  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:26:31:a4  txqueuelen 1000  (Ethernet)
        RX packets 124344  bytes 49376281 (47.0 MiB)
```

绑定虚拟网卡到 `ens33`

```bash
ifconfig ens33:1 192.168.194.200/24
```

```bash
ifconfig

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.194.128  netmask 255.255.255.0  broadcast 192.168.194.255

ens33:1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.194.200  netmask 255.255.255.0  broadcast 192.168.194.255
```

### 3.6 启动 HAProxy

```bash
haproxy -f /etc/haproxy/haproxy.cfg
```

查看端口是否启动成功：

```bash
netstat -nltp

# 启动成功
tcp      0     0 192.168.194.200:8096     0.0.0.0:*        LISTEN      14083/haproxy
tcp      0     0 192.168.194.200:48800    0.0.0.0:*        LISTEN      14083/haproxy
tcp      0     0 192.168.194.200:8097     0.0.0.0:*        LISTEN      14083/haproxy
```

## 4. 用同样的方法配置 192.168.194.131 节点

## 5. 登录 MyCAT 验证配置

131 主机节点

```bash
mysql -uapp_koax -p -h192.168.194.200 -P8096 -A

# 登录成功
Welcome to the MySQL monitor.  Commands end with ; or \g.
```
