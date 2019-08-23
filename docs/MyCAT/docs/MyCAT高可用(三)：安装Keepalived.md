# 安装 Keepalived

- 在 `128` 和 `131` 节点中安装 Keepalived

- Keepalived 用来管理配置的虚拟 IP `192.168.194.200`

## 1. yum 安装 Keepalived

```bash
yum install keepalived -y
```

### 1.1 配置 keepalived.conf

[`keepalived.conf` 此处下载](../file/keepalived.conf)

```bash
vi /etc/keepalived/keepalived.conf
```

```yml
! Configuration Fileforkeepalived

global_defs {
    #一个没重复的名字即可
    router_id xxoo_master
}

# 设置运行检测的脚本文件
vrrp_script chk_http_port {
script"/etc/keepalived/check_haproxy.sh"
  #每隔2秒执行一次
  interval 2
  weight 2
}

vrrp_instance VI_1 {
  # 设置主节点节点MASTER
  state MASTER

  # 当前所用的网卡(ifconfig或ip addr 查看网卡)
  interface ens33

  # 同一个keepalived集群的virtual_router_id相同
  virtual_router_id 51

  # 权重，master要大于slave
  priority 150

  # 主备通讯时间间隔
  advert_int 1

  # 设置nopreempt防止抢占资源
  nopreempt

  # 主备保持一致
  authentication {
    auth_type PASS
    auth_pass 1111
  }

  # 与上方设置运行检测文件的呼应
  track_script {
    chk_http_port
  }

  virtual_ipaddress {
    # 虚拟ip地址（VIP，一个尚未占用的内网ip即可）
    192.168.194.200 dev ens33 scope global
  }

}
```

### 1.2 添加检查 check_haproxy.sh 脚本文件

[`check_haproxy.sh` 此处下载](../file/check_haproxy.sh)

```bash
vi /etc/keepalived/check_haproxy.sh

# 添加权限
chmod a+x check_haproxy.sh
```

```yml
#!/bin/bash
STARTHAPROXY="/usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg"
#STOPKEEPALIVED="/etc/init.d/keepalived stop"
STOPKEEPALIVED="/usr/bin/systemctl stop keepalived"
LOGFILE="/var/log/keepalived-haproxy-state.log"
echo "[check_haproxy status]" >> $LOGFILE
A=`ps -C haproxy --no-header |wc -l`
echo "[check_haproxy status]" >> $LOGFILE
date >> $LOGFILE
if [ $A -eq 0 ];then
   echo $STARTHAPROXY >> $LOGFILE
   $STARTHAPROXY >> $LOGFILE 2>&1
   sleep 5
fi
if [ `ps -C haproxy --no-header |wc -l` -eq 0 ];then
   exit 0
else
   exit 1
fi
```

### 1.3 启动 keepalived

```bash
systemctl start keepalived.service
```

查看是否启动成功：

```bash
ps -ef | grep keepalive

root     119263        1  0 04:38 ?        00:00:00 /usr/sbin/keepalived -D
root     119264   119263  0 04:38 ?        00:00:00 /usr/sbin/keepalived -D
root     119265   119263  0 04:38 ?        00:00:00 /usr/sbin/keepalived -D
root     119392     1214  0 04:38 pts/0    00:00:00 grep --color=auto keepalive
```

```bash
ip addr

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:26:31:a4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.194.128/24 brd 192.168.194.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet 192.168.194.200/32 scope global ens33
       valid_lft forever preferred_lft forever
    inet 192.168.194.200/24 brd 192.168.194.255 scope global secondary ens33:1
```

- HAProxy 绑定的为 `192.168.194.200/24`

- Keepalive 绑定的为 `192.168.194.200/32`

## 2. 按照上述方法配置 131 节点

131 设置为 `slave` 节点

### 2.1 配置 keepalived.conf

[`keepalived.conf` 此处下载](../file/keepalived.conf)

```bash
vi /etc/keepalived/keepalived.conf
```

```yml
vrrp_script chk_http_port {
script"/etc/keepalived/check_haproxy.sh"
  interval 2
  weight 2
}
vrrp_instance VI_1 {
  # 此处不设置为MASTER，通过priority来竞争master
  state BACKUP
  interface ens33
  virtual_router_id 51

  # 权重低于master节点
  priority 120
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  track_script {
    chk_http_port
  }
  virtual_ipaddress {
    192.168.194.200 dev ens33 scope global
  }
}
```

其他配置基本一致。

### 2.2 启动 131 节点 keepalived

---

## 3. 高可用测试

### 3.1 关闭 master 的 keepalived

```bash
systemctl stop keepalived.service
```

```bash
ip addr

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:26:31:a4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.194.128/24 brd 192.168.194.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet 192.168.194.200/24 brd 192.168.194.255 scope global secondary ens33:1
```

> 没有了绑定的虚拟IP：`inet 192.168.194.200/32 scope global ens33`

### 3.2 查看 slave 节点的 keepalived

虚拟 IP 绑定到子节点上：

```bash
ip addr

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:a5:09:3f brd ff:ff:ff:ff:ff:ff
    inet 192.168.194.131/24 brd 192.168.194.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet 192.168.194.200/32 scope global ens33
       valid_lft forever preferred_lft forever
```

高可用测试成功。
