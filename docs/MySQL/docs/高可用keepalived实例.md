# 高可用keepalived实例

![iamge](../images/03.png)

主服务器宕机，需要手动切换从服务器，切换中间耗时很长，业务中断不能忍受。

**虚拟IP（vip）：**为了解决手动切换实现高可用问题，给一个未分配给真实主机的 IP，也就是说对外提供服务器的主机除了有一个真是 IP 外，还有一个虚 IP。

本例使用 keepalived 配置 vip。

引入 VIP 后的数据库架构

![iamge](../images/04.png)

## mysql+keepalived实现双主自由切换

![iamge](../images/05.png)

### 演示环境说明

| 主机名 | IP | 角色 |
| ---- | ---- | ---- |
| localhost151 | 192.168.194.151 | 主DB |
| localhost152 | 192.168.194.152 | 主备DB |
| keepalived | 192.168.194.100 | 虚ip地址 |

保证只有一个主提供服务，另一个提供只读的服务。

### 主 数据库配置修改

```yml
# my.cnf
#控制自增id增长的步长，一次增长2
auto_increment_increment=2

#自增id从哪个之开始增长
auto_increment_offset=1
```

```sql
# mysql>
set global auto_increment_increment=2;
set global auto_increment_offset=1;
```

### 主备 数据库配置修改

```sql
# my.cnf
auto_increment_increment=2

auto_increment_offset=2
```

```sql
# mysql>
set global auto_increment_increment=2;
set global auto_increment_offset=2;
```

```sql
mysql> show master status \G
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 154
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```

在主服务上执行：

```sql
change master to 
master_host='slave_host_ip',
master_user = 'replname',
master_password='password',
master_log_file='mysql-bin.000001',
master_log_pos=154;
```

启动主 slave:

```sql
start slave;
```

---

## Keepalived 安装

![iamge](../images/06.png)

### 配置主服务器 keepalived.cnf

```yml
! Configuration File for keepalived

global_defs {
   router_id mysql_ha
}
vrrp_script check_run {
    script "/etc/keepalived/check_mysql.sh"
    interval 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 201
    priority 99
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1200
    }

  track_script {
       check_run
     }

   virtual_ipaddress {
      192.168.194.100/24 dev ens33 scope global
    }
}
```

`check_mysql.sh` 检测脚本

网上很多脚本执行错误，这个简陋版的起码不会报错

```bash
#!/bin/bash
#This scripts is check for Mysql Slave status
counter=$(netstat -na|grep "LISTEN"|grep "3306"|wc -l)
if [ "${counter}" -eq 0 ]; then
    systemctl stop keepalived
    killall keepalived
fi
```

### 配置主备服务器 keepalived.cnf

```yml
! Configuration File for keepalived

global_defs {
   router_id mysql_ha
}
vrrp_script check_run {
    script "/etc/keepalived/check_mysql.sh"
    interval 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 201
    priority 99
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1200
    }

  track_script {
       check_run
     }

   virtual_ipaddress {
      192.168.194.100/24 dev ens33 scope global
    }
}
```

`check_mysql.sh` 检测脚本

## 执行 keepalived

双主库都执行

```bash
systemctl start keepalived.service
```

查看 master1 VIP绑定成功 ：

```bash
ip addr

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:af:15:4f brd ff:ff:ff:ff:ff:ff
    inet 192.168.194.151/24 brd 192.168.194.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet 192.168.194.100/24 scope global secondary ens33
       valid_lft forever preferred_lft forever
```

关闭 master1 mysql服务：

```bash
systemctl stop mysqld.service
```

```bash
ip addr

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:af:15:4f brd ff:ff:ff:ff:ff:ff
    inet 192.168.194.151/24 brd 192.168.194.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
```

查看 Master2 跳转绑定 VIP 成功 :

```bash
ip addr

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:4e:8b:98 brd ff:ff:ff:ff:ff:ff
    inet 192.168.194.152/24 brd 192.168.194.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet 192.168.194.100/24 scope global secondary ens33
       valid_lft forever preferred_lft forever
```
